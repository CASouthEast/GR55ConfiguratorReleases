# GR-55 SysEx Parsing Guide

Complete, non-scanned SysEx parameter reference for the Roland GR-55, refined from the MIDI Implementation address map (`gr55_MIDI.pdf`), the owner’s manual, and the live app Data JSON. Use this together with `GR55Configurator/Data/*.json` (addresses the app actually uses) and the OM MD docs (semantics).



## Status and sources

| Concern | Winner |
|---|---|
| App dump byte indices / lengths | Live `GR55Configurator/Data/*.json` + parsers |
| Printed MIDI map addresses, bit masks, enum order | [`gr55_MIDI.pdf`](./gr55_MIDI.pdf) |
| Meanings / UI constraints | OM MD set ([Effects_Index.md](./Effects_Index.md), [Settings_Index.md](./Settings_Index.md), …) |

### Address-gap policy

The MIDI Implementation map jumps (e.g. `00 7F` → `01 00`). That is **not** how Temporary Patch dump bytes are indexed. Tables that list both columns use **App byte index** for dumps and **MIDI map** for the PDF. Prefer Data JSON when in doubt; do not invent new compensation math here.

### MIDI Implementation row schema

Parameter tables use one shared Markdown shape (same as **Patch Common**):

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |

1. **App byte index** — dump / Temporary Patch index the app uses
2. **MIDI map** — address printed in `gr55_MIDI.pdf` (same as app index when there is no gap)
3. **Parameter** — name
4. **Stored (SysEx)** — wire range
5. **Presentation / notes** — enums, signed ranges, L↔R, etc.; bit pattern (`0000 aaaa`) folded here when known
6. Optional footnote (`*C127`, …) — see MIDI PDF end-matter / JSON `encoding`

## 1. SysEx Message Structure

A standard Roland GR-55 SysEx message follows this format:

```
F0 41 10 00 00 53 12 [Address] [Data] [Checksum] F7
```

| Component | Length | Description |
|-----------|--------|-------------|
| **SOX** | 1 byte | `F0` (Start of Exclusive) |
| **Manufacturer** | 1 byte | `41` (Roland) |
| **Device ID** | 1 byte | `10` (Default, configurable) |
| **Model ID** | 4 bytes | `00 00 53` (GR-55) |
| **Command** | 1 byte | `12` (DT1 - Data Set) or `11` (RQ1 - Request Data) |
| **Address** | 4 bytes | The starting address of the data (BIG Endian). **NOT** 7-bit packed in the header. |
| **Data** | N bytes | The payload. **IS** 7-bit packed (each byte 0x00-0x7F). |
| **Checksum** | 1 byte | Validation checksum. |
| **EOX** | 1 byte | `F7` (End of Exclusive) |

### 1.1 Address & Offset Calculation

The **Address** in the header defines where the **Data** block starts in the device's memory map.

To map a specific parameter to the data bytes:
1.  **Base Address**: The starting address from the SysEx Header.
2.  **Parameter Address**: The absolute address of the parameter (defined in JSON).
3.  **Byte Index**: `Parameter Address - Base Address`.

> **Note**: For "Common" patches and most bulk dumps, the Base Address is usually `18 00 00 00` (Temporary Patch) or similar. If the Base Address is `00 00 00 00` (relative), the Offset equals the Byte Index.

### 1.2 Message base addresses / block ids

Top-level absolute SysEx roots (4-byte Roland address). App: `GR55SystemAddressMap`, Temporary Patch `18 00 00 00`.

| Absolute | Block | Notes |
| :--- | :--- | :--- |
| **`01 00 00 00`** | **Setup** | Current patch Bank MSB + PC (`01 00 00 00` / `01 00 00 01`). See §6. |
| **`02 00 00 00`** | **System** (root) | Global settings. Sub-blocks below; detail in §5. |
| **`02 00 00 00`** | System Common | Output, MIDI, tuner, RX MAP Select, … |
| **`02 00 02 00`** | System Pedal / CTL–EXP–GK | Pedal assign defaults |
| **`02 00 04 00`…`02 00 0D 00`** | System GK SET 1–10 | Pickup sets |
| **`02 01 00 00`** | Guitar Rx PC MAP bank 0 | Banks 1…63 stride `04 00` (see §5.1) |
| **`02 03 00 00`** | Bass Rx PC MAP bank 0 | Same stride pattern under `02 03` / `02 04` |
| **`18 00 00 00`** | **Temporary Patch** (current) | Live edit buffer; section offsets below |
| **`20` / `21` / `22` …** | USER patch slots | Stored USER memory (WRITE / EXCHANGE / INITIALIZE) |

#### Temporary Patch section offsets (under `18 00 00 00`)

Absolute address = `18 00 00 00` + offset (e.g. MFX → `18 00 03 00`).

| Offset | MIDI map | Block |
| :--- | :--- | :--- |
| **00 00 00** | 00 00 00 | Patch Common |
| **00 03 00** | 00 03 00 | Patch MFX |
| **00 06 00** | 00 06 00 | Patch CHO / DLY / REV / EQ / CHAR |
| **00 07 00** | 00 07 00 | Patch AMP / MOD / NS |
| **00 10 00** | 00 10 00 | Patch Modeling Tone |
| **00 20 00** | 00 20 00 | Patch PCM 1 Tone Part 1 |
| **00 21 00** | 00 21 00 | Patch PCM 2 Tone Part 1 |
| **00 30 00** | 00 30 00 | Patch PCM 1 Tone Part 2 |
| **00 31 00** | 00 31 00 | Patch PCM 2 Tone Part 2 |

---

## 2. Data Types & Extraction Logic

GR-55 data is 7-bit packed standard MIDI, but specific fields often span multiple bytes using nibble-packing.

### 2.1 Standard 7-bit Byte (`UByte`, `value`)
A single byte value ranging from 0-127.
-   **Size**: 1 Byte
-   **Range**: 0-127 (`0x00`–`0x7F`)
-   **App**: `BitPackedFieldExtractor.extractUByte` / `SysExDataExtractor.extractUByte`
-   **Extraction**:
    ```javascript
    const value = dataBytes[index] & 0x7F;
    ```
    ```swift
    let value = Int(dataBytes[index] & 0x7F)
    ```

### 2.2 Split 8-bit (`USplit8`)
An 8-bit value (0–255) across 2 bytes (low 4 bits of each).
-   **Size**: 2 Bytes
-   **Range**: 0-255
-   **Format (dump helper)**: `[0000 LLLL] [0000 HHHH]` — **low nibble first**, then high
-   **App**: `BitPackedFieldExtractor.extractUSplit8`
-   **Note**: Some encode paths (e.g. `MIDIValueMapper` `.nibble2` for Patch Level/Tempo) use **high-then-low**. Prefer the field’s catalog/encoder, not this helper alone.
-   **Extraction**:
    ```javascript
    const low = dataBytes[index] & 0x0F;
    const high = dataBytes[index + 1] & 0x0F;
    const value = (high << 4) | low;
    ```
    ```swift
    let low = Int(dataBytes[index] & 0x0F)
    let high = Int(dataBytes[index + 1] & 0x0F)
    let value = (high << 4) | low
    ```

### 2.3 Split 12-bit (`USplit12`)
A 12-bit value (0–4095) across 3 bytes. Used for wide ranges (delay time, assign Target Min/Max wire, etc.).
-   **Size**: 3 Bytes
-   **Range**: 0-4095
-   **Format**: `[0000 HHHH] [0000 MMMM] [0000 LLLL]` — **MSB-first** (high, mid, low)
-   **App**: `BitPackedFieldExtractor.extractUSplit12` / `USplit12Field.decodeValue` / `MIDIValueMapper` `.nibble3`
-   **Extraction**:
    ```javascript
    const high = dataBytes[index] & 0x0F;
    const mid  = dataBytes[index + 1] & 0x0F;
    const low  = dataBytes[index + 2] & 0x0F;
    const value = (high << 8) | (mid << 4) | low;
    ```
    ```swift
    let high = Int(dataBytes[index] & 0x0F)
    let mid  = Int(dataBytes[index + 1] & 0x0F)
    let low  = Int(dataBytes[index + 2] & 0x0F)
    let value = (high << 8) | (mid << 4) | low
    ```
-   **Assign Target Min/Max (signed display)**: **not** 2’s complement. Wire is USplit12 with `encodedOffset` **1024**; display = `wire − 1024` (−1024…+1023). App: `extractAssignTargetMinMax` (encode adds +1024 via `HWParameterRegistry` / `MIDIValueMapper`).

### 2.4 Assign Target Key (Special 3-Byte)
Used for Assign Targets (mappings).
-   **Size**: 3 Bytes
-   **Format**:
    -   Byte 0 low nibble: Level 1 category (`00`, `01`, `02`…)
    -   Byte 1 low nibble: high digit of Level 2
    -   Byte 2 low nibble: low digit of Level 2
-   **App**: `BitPackedFieldExtractor.extractAssignTargetKey`
-   **Extraction**:
    ```javascript
    const level1 = dataBytes[index] & 0x0F;
    const hex1 = dataBytes[index + 1] & 0x0F;
    const hex2 = dataBytes[index + 2] & 0x0F;
    const level2 = (hex1 << 4) | hex2;
    const key = `${level1.toString(16).padStart(2, '0').toUpperCase()}-${level2.toString(16).padStart(2, '0').toUpperCase()}`;
    ```
    ```swift
    let level1 = Int(dataBytes[index] & 0x0F)
    let hex1 = Int(dataBytes[index + 1] & 0x0F)
    let hex2 = Int(dataBytes[index + 2] & 0x0F)
    let key = String(format: "%02X-%02X", level1, (hex1 << 4) | hex2)  // e.g. "00-73"
    ```

### 2.5 Boolean / Bit Flags
Single bits packed into a byte.
-   **App**: `BitPackedFieldExtractor.extractBoolean` / `extractBits`
-   **Extraction**:
    ```javascript
    const bitPosition = 0; // 0-7
    const isSet = (dataBytes[index] & (1 << bitPosition)) !== 0;
    ```
    ```swift
    let bitPosition = 0 // 0-7
    let isSet = (dataBytes[index] & (1 << bitPosition)) != 0
    ```

---

## 3. Parameter Mapping (JSON to Logic)

The project uses JSON files (e.g. `Patch_Assigns.json`) to define dump-parse parameters. Live HW edit / tree writes use `SharedParamCatalog` + `MIDIValueMapper` instead; this section is the **dump → model** path (`AssignsDataParser`, `GeneralDataParser`, …).

### JSON Field Definitions
```json
{
  "name": "ASSIGN1 Target Min",
  "address": "90",
  "type": "value",
  "range": { "min": -1024, "max": 1023 },
  "length": 3
}
```

### Mapping Rules

1.  **Address → byte index**:
    -   JSON `address` is a hex offset (often already an **app dump index** within the block).
    -   `parseInt("90", 16)` = 144.
    -   App: `AddressToByteIndexCalculator.byteIndex(offset:baseAddress:)` → `offset` when `baseAddress == 0`, else `offset - baseAddress`.
    -   **MFX** MIDI-map offsets `≥ 0x0100` also need `MfxFieldTypeHelpers.correctedByteIndex` (`− 0x80`) before indexing the dump.

2.  **Type determination**:
    -   **`length: 3`** + **`type: "value"`**: usually **`USplit12`** (MSB-first).
    -   **`length: 2`** + **`type: "value"`**: usually **`USplit8`** (low-then-high helper).
    -   **`length: 1` (or omitted)** + **`type: "value"`**: **`UByte`**.
    -   **`type: "options"`**: **`UByte`**, matched to the option list.

3.  **Assign special cases (by name)** — see `AssignsDataParser.parseValueParameter`:
    -   `"Target"` (not Min/Max), length 3 → **Assign Target Key**.
    -   `"Target Min"` / `"Target Max"` → **`extractAssignTargetMinMax`** (`USplit12 − 1024`), not 2’s complement.
    -   `"Wave Pedal Rate"` → raw **`USplit12`**.

### JavaScript (illustrative dump parse)

```javascript
function byteIndex(offset, baseAddress = 0) {
    return baseAddress === 0 ? offset : offset - baseAddress;
}

function extractUByte(dataBytes, index) {
    return dataBytes[index] & 0x7F;
}

function extractUSplit8(dataBytes, index) {
    const low = dataBytes[index] & 0x0F;
    const high = dataBytes[index + 1] & 0x0F;
    return (high << 4) | low;
}

function extractUSplit12(dataBytes, index) {
    // MSB-first (matches BitPackedFieldExtractor / Delay / nibble3)
    const high = dataBytes[index] & 0x0F;
    const mid  = dataBytes[index + 1] & 0x0F;
    const low  = dataBytes[index + 2] & 0x0F;
    return (high << 8) | (mid << 4) | low;
}

function extractAssignTargetMinMax(dataBytes, index) {
    return extractUSplit12(dataBytes, index) - 1024;
}

function parseParameter(paramDef, dataBuffer, baseAddress = 0) {
    const offset = parseInt(paramDef.address, 16);
    const index = byteIndex(offset, baseAddress);

    if (paramDef.length === 3) {
        if (paramDef.name.includes("Target")
            && !paramDef.name.includes("Min")
            && !paramDef.name.includes("Max")) {
            return extractAssignTargetKey(dataBuffer, index);
        }
        if (paramDef.name.includes("Target Min") || paramDef.name.includes("Target Max")) {
            return extractAssignTargetMinMax(dataBuffer, index);
        }
        return extractUSplit12(dataBuffer, index);
    }
    if (paramDef.length === 2) {
        return extractUSplit8(dataBuffer, index);
    }
    return extractUByte(dataBuffer, index);
}
```

### Swift equivalent (app dump path)

Mirrors `AddressToByteIndexCalculator` + `BitPackedFieldExtractor` as used by `AssignsDataParser`:

```swift
func byteIndex(offset: Int, baseAddress: Int = 0) -> Int {
    baseAddress == 0 ? offset : offset - baseAddress
}

func parseParameter(
    name: String,
    addressHex: String,
    length: Int?,
    dataBytes: [UInt8],
    baseAddress: Int = 0
) -> Any {
    let offset = Int(addressHex, radix: 16) ?? 0
    let index = byteIndex(offset: offset, baseAddress: baseAddress)

    if length == 3 {
        if name.contains("Target"), !name.contains("Min"), !name.contains("Max") {
            return BitPackedFieldExtractor.extractAssignTargetKey(dataBytes, index: index)
        }
        if name.contains("Target Min") || name.contains("Target Max") {
            return BitPackedFieldExtractor.extractAssignTargetMinMax(dataBytes, index: index)
        }
        return BitPackedFieldExtractor.extractUSplit12(dataBytes, index: index)
    }
    if length == 2 {
        return BitPackedFieldExtractor.extractUSplit8(dataBytes, index: index)
    }
    return BitPackedFieldExtractor.extractUByte(dataBytes, index: index)
}
```

Live parameter **writes** (HW Edit / tree) do not use this dump loop: they go `SharedParamCatalog` → `HWParameterRegistry.encodedWrites` → `MIDIValueMapper.encode` → DT1.

---

### Examples

#### Example 1: Assign 1 Target Min
-   **JSON**: `Address: 90`, `Length: 3`, `Range: -1024 to 1023`
-   **Bytes**: `0E 00 00` (example wire)
-   **Parsing**:
    1.  Read 3 bytes at offset `0x90`.
    2.  `extractUSplit12` → wire value.
    3.  Display = `wire − 1024` (`extractAssignTargetMinMax`).

#### Example 2: Assign 1 Switch
-   **JSON**: `Address: 8C`, `Type: Options` (Off/On)
-   **Bytes**: `01`
-   **Parsing**:
    1.  Read 1 byte at Offset 0x8C.
    2.  Value `0x01` corresponds to Option `01` ("ON").

---

## 4. Patch SysEx (Temporary Patch)

Parameter maps for the live Temporary Patch area (`18 00 00 00` + section offsets from §1.2). Row shape matches Patch Common (§ MIDI Implementation row schema).

### 4.0 Patch Common

Complete **Patch Common** parameter list. **App byte index** is what Temporary Patch dumps use (Roland’s printed map skips `00 7F`→`01 00` and `01 7F`→`02 00`; the app compensates). **MIDI map** is the address printed in `gr55_MIDI.pdf`.

Base: Temporary Patch **`18 00 00 00`**. Live JSON: `Patch_General.json`, `Patch_CTL.json`, `Patch_EXP.json`, `Patch_GK.json`, `Patch_Assigns.json`, `Patch_Master.json`. Semantics: [MASTER.md](./MASTER.md), [ASSIGNS.md](./ASSIGNS.md).

#### Patch Common parameters

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **Patch Attribute** | 0 - 1 | 0=GUITAR, 1=BASS |
| **00 01 - 00 10** | 00 01 - 00 10 | **Patch Name 1-16** | 32 - 127 | 16 ASCII characters |
| **00 11** | 00 11 | **CTL Status** | 0 - 1 | 0=OFF, 1=ON |
| **00 12** | 00 12 | **CTL Function** | 0 - 16 | OFF, HOLD, TAP, TONE SW, AMP, etc. |
| **00 13** | 00 13 | **CTL Hold Type** | 0 - 3 | 1 - 4 |
| **00 14** | 00 14 | **CTL Hold Sw Mode** | 0 - 1 | 0=LATCH, 1=MOMENT |
| **00 15** | 00 15 | **CTL Hold PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 16** | 00 16 | **CTL Hold PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 17** | 00 17 | **CTL=OFF PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 18** | 00 18 | **CTL=OFF PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 19** | 00 19 | **CTL=OFF Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 1A** | 00 1A | **CTL=OFF Normal PU** | 0 - 1 | 0=OFF, 1=ON |
| **00 1B** | 00 1B | **CTL=ON PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 1C** | 00 1C | **CTL=ON PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 1D** | 00 1D | **CTL=ON Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 1E** | 00 1E | **CTL=ON Normal PU** | 0 - 1 | 0=OFF, 1=ON |
| **00 1F** | 00 1F | **EXP PDL(OFF) Func** | 0 - 9 | Vol, Pitch, Mod, etc. |
| **00 20** | 00 20 | **EXP(OFF) Vol PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 21** | 00 21 | **EXP(OFF) Vol PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 22** | 00 22 | **EXP(OFF) Vol Model** | 0 - 1 | 0=OFF, 1=ON |
| **00 23** | 00 23 | **EXP(OFF) Vol Norm** | 0 - 1 | 0=OFF, 1=ON |
| **00 24** | 00 24 | **EXP(OFF) Bend Range** | 12 - 36 | -12 to +12 semitones |
| **00 25** | 00 25 | **EXP(OFF) Bend PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 26** | 00 26 | **EXP(OFF) Bend PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 27** | 00 27 | **EXP(OFF) Bend Model** | 0 - 1 | 0=OFF, 1=ON |
| **00 28** | 00 28 | **EXP(OFF) Mod Min** | 0 - 127 | Skalning *C127 |
| **00 29** | 00 29 | **EXP(OFF) Mod Max** | 0 - 127 | Skalning *C127 |
| **00 2A** | 00 2A | **EXP(OFF) Mod PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 2B** | 00 2B | **EXP(OFF) Mod PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 2C** | 00 2C | **EXP(OFF) X-Fade P1** | 0 - 2 | OFF, TOE, HEEL |
| **00 2D** | 00 2D | **EXP(OFF) X-Fade P2** | 0 - 2 | OFF, TOE, HEEL |
| **00 2E** | 00 2E | **EXP(OFF) X-Fade Mod** | 0 - 2 | OFF, TOE, HEEL |
| **00 2F** | 00 2F | **EXP(OFF) X-Fade Nor** | 0 - 2 | OFF, TOE, HEEL |
| **00 30** | 00 30 | **EXP(OFF) Dly Min** | 0 - 120 | 0 - 120 |
| **00 31** | 00 31 | **EXP(OFF) Dly Max** | 0 - 120 | 0 - 120 |
| **00 32** | 00 32 | **EXP(OFF) Rev Min** | 0 - 100 | 0 - 100 |
| **00 33** | 00 33 | **EXP(OFF) Rev Max** | 0 - 100 | 0 - 100 |
| **00 34** | 00 34 | **EXP(OFF) Cho Min** | 0 - 100 | 0 - 100 |
| **00 35** | 00 35 | **EXP(OFF) Cho Max** | 0 - 100 | 0 - 100 |
| **00 36** | 00 36 | **EXP PDL(ON) Func** | 0 - 9 | Vol, Pitch, Mod, etc. |
| **00 37** | 00 37 | **EXP(ON) Vol PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 38** | 00 38 | **EXP(ON) Vol PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 39** | 00 39 | **EXP(ON) Vol Model** | 0 - 1 | 0=OFF, 1=ON |
| **00 3A** | 00 3A | **EXP(ON) Vol Normal** | 0 - 1 | 0=OFF, 1=ON |
| **00 3B** | 00 3B | **EXP(ON) Bend Range** | 12 - 36 | -12 to +12 semitones |
| **00 3C** | 00 3C | **EXP(ON) Bend PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 3D** | 00 3D | **EXP(ON) Bend PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 3E** | 00 3E | **EXP(ON) Bend Model** | 0 - 1 | 0=OFF, 1=ON |
| **00 3F** | 00 3F | **EXP(ON) Mod Min** | 0 - 127 | Skalning *C127 |
| **00 40** | 00 40 | **EXP(ON) Mod Max** | 0 - 127 | Skalning *C127 |
| **00 41** | 00 41 | **EXP(ON) Mod PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 42** | 00 42 | **EXP(ON) Mod PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 43** | 00 43 | **EXP(ON) X-Fade P1** | 0 - 2 | OFF, TOE, HEEL |
| **00 44** | 00 44 | **EXP(ON) X-Fade P2** | 0 - 2 | OFF, TOE, HEEL |
| **00 45** | 00 45 | **EXP(ON) X-Fade Mod** | 0 - 2 | OFF, TOE, HEEL |
| **00 46** | 00 46 | **EXP(ON) X-Fade Nor** | 0 - 2 | OFF, TOE, HEEL |
| **00 47** | 00 47 | **EXP(ON) Dly Min** | 0 - 120 | 0 - 120 |
| **00 48** | 00 48 | **EXP(ON) Dly Max** | 0 - 120 | 0 - 120 |
| **00 49** | 00 49 | **EXP(ON) Rev Min** | 0 - 100 | 0 - 100 |
| **00 4A** | 00 4A | **EXP(ON) Rev Max** | 0 - 100 | 0 - 100 |
| **00 4B** | 00 4B | **EXP(ON) Cho Min** | 0 - 100 | 0 - 100 |
| **00 4C** | 00 4C | **EXP(ON) Cho Max** | 0 - 100 | 0 - 100 |
| **00 4D** | 00 4D | **EXP SW Status** | 0 - 1 | 0=OFF, 1=ON |
| **00 4E** | 00 4E | **EXP SW Function** | 0 - 13 | OFF, TAP, TONE, etc. |
| **00 4F - 00 52**| 00 4F - 00 52 | **(Reserved)** | 0 - 3 | Reserved |
| **00 53** | 00 53 | **EXP SW=OFF PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 54** | 00 54 | **EXP SW=OFF PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 55** | 00 55 | **EXP SW=OFF Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 56** | 00 56 | **EXP SW=OFF Norm Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 57** | 00 57 | **EXP SW=ON PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 58** | 00 58 | **EXP SW=ON PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 59** | 00 59 | **EXP SW=ON Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 5A** | 00 5A | **EXP SW=ON Normal Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 5B** | 00 5B | **GK VOL Function** | 0 - 9 | Vol, Pitch, Mod, etc. |
| **00 5C** | 00 5C | **GK VOL PCM1 Vol** | 0 - 1 | 0=OFF, 1=ON |
| **00 5D** | 00 5D | **GK VOL PCM2 Vol** | 0 - 1 | 0=OFF, 1=ON |
| **00 5E** | 00 5E | **GK VOL Model Vol** | 0 - 1 | 0=OFF, 1=ON |
| **00 5F** | 00 5F | **GK VOL Normal Vol** | 0 - 1 | 0=OFF, 1=ON |
| **00 60** | 00 60 | **GK VOL Bend Range** | 12 - 36 | -12 to +12 semitones |
| **00 61** | 00 61 | **GK VOL Bend PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 62** | 00 62 | **GK VOL Bend PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 63** | 00 63 | **GK VOL Bend Model** | 0 - 1 | 0=OFF, 1=ON |
| **00 64** | 00 64 | **GK VOL Mod Min** | 0 - 127 | Skalning *C127 |
| **00 65** | 00 65 | **GK VOL Mod Max** | 0 - 127 | Skalning *C127 |
| **00 66** | 00 66 | **GK VOL Mod PCM1** | 0 - 1 | 0=OFF, 1=ON |
| **00 67** | 00 67 | **GK VOL Mod PCM2** | 0 - 1 | 0=OFF, 1=ON |
| **00 68** | 00 68 | **GK VOL X-Fade P1** | 0 - 2 | OFF, TOE, HEEL |
| **00 69** | 00 69 | **GK VOL X-Fade P2** | 0 - 2 | OFF, TOE, HEEL |
| **00 6A** | 00 6A | **GK VOL X-Fade Mod** | 0 - 2 | OFF, TOE, HEEL |
| **00 6B** | 00 6B | **GK VOL X-Fade Nor** | 0 - 2 | OFF, TOE, HEEL |
| **00 6C** | 00 6C | **GK VOL Dly Min** | 0 - 120 | 0 - 120 |
| **00 6D** | 00 6D | **GK VOL Dly Max** | 0 - 120 | 0 - 120 |
| **00 6E** | 00 6E | **GK VOL Rev Min** | 0 - 100 | 0 - 100 |
| **00 6F** | 00 6F | **GK VOL Rev Max** | 0 - 100 | 0 - 100 |
| **00 70** | 00 70 | **GK VOL Cho Min** | 0 - 100 | 0 - 100 |
| **00 71** | 00 71 | **GK VOL Cho Max** | 0 - 100 | 0 - 100 |
| **00 72** | 00 72 | **GK S1 Function** | 0 - 13 | OFF, TAP, TONE, etc. |
| **00 73 - 00 76**| 00 73 - 00 76 | **(Reserved)** | 0 - 3 | Reserved |
| **00 77** | 00 77 | **GK S1=OFF PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 78** | 00 78 | **GK S1=OFF PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 79** | 00 79 | **GK S1=OFF Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7A** | 00 7A | **GK S1=OFF Norm Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7B** | 00 7B | **GK S1=ON PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7C** | 00 7C | **GK S1=ON PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7D** | 00 7D | **GK S1=ON Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7E** | 00 7E | **GK S1=ON Norm Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 7F** | 00 7F | **GK S2 Function** | 0 - 13 | OFF, TAP, TONE, etc. |
| **00 80** | 01 00 | **(Reserved)** | 0 - 3 | Reserved |
| **00 84** | 01 04 | **GK S2=OFF PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 85** | 01 05 | **GK S2=OFF PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 86** | 01 06 | **GK S2=OFF Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 87** | 01 07 | **GK S2=OFF Norm Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 88** | 01 08 | **GK S2=ON PCM1 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 89** | 01 09 | **GK S2=ON PCM2 Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 8A** | 01 0A | **GK S2=ON Model Sw** | 0 - 1 | 0=OFF, 1=ON |
| **00 8B** | 01 0B | **GK S2=ON Normal Sw** | 0 - 1 | 0=OFF, 1=ON |

---

#### Assigns (Assign 1-8)
Each Assign block has the same structure. **Assign 1** is shown below; later assigns add their respective offsets.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 8C** | 01 0C | **ASSIGN1 Switch** | 0 - 1 | 0=OFF, 1=ON |
| **00 8D** | 01 0D | **ASSIGN1 Target** | 0 - 1024 | **Nibble (*2)** |
| **00 90** | 01 10 | **ASSIGN1 Target Min**| 0 - 2047 | **Nibble (*2)** (-1024 - +1023) |
| **00 93** | 01 13 | **ASSIGN1 Target Max**| 0 - 2047 | **Nibble (*2)** (-1024 - +1023) |
| **00 96** | 01 16 | **ASSIGN1 Source** | 0 - 71 | CTL, EXP, GK, MIDI CC, etc. |
| **00 97** | 01 17 | **ASSIGN1 Src Mode** | 0 - 1 | 0=MOMENT, 1=TOGGLE |
| **00 98** | 01 18 | **ASSIGN1 Range Lo** | 0 - 126 | Active range low |
| **00 99** | 01 19 | **ASSIGN1 Range Hi** | 1 - 127 | Active range high |
| **00 9A** | 01 1A | **ASSIGN1 Trigger** | 0 - 10 | Patch Chg, CTL, EXP, etc. |
| **00 9B** | 01 1B | **ASSIGN1 Time** | 0 - 100 | Internal pedal time |
| **00 9C** | 01 1C | **ASSIGN1 Curve** | 0 - 2 | Linear, Slow, Fast |
| **00 9D** | 01 1D | **ASSIGN1 Wave Rate** | 0 - 113 | Rate / note value |
| **00 9E** | 01 1E | **ASSIGN1 Wave Form** | 0 - 2 | SAW, TRI, SIN |

*   **Assign 2** starts at app index **00 9F** (MIDI map `01 1F`).
*   **Assign 3** starts at app index **00 B2** (MIDI map `01 32`).
*   **Assign 4** starts at app index **00 C5** (MIDI map `01 45`).
*   **Assign 5** starts at app index **00 D8** (MIDI map `01 58`).
*   **Assign 6** starts at app index **00 EB** (MIDI map `01 6B`).
*   **Assign 7** starts at app index **00 FE** (MIDI map `01 7E`).
    *   *Note: From Assign 7 Target onward the MIDI map jumps to `02 02`. App index = MIDI − `0x100` → **01 02**.*
*   **Assign 8** starts at app index **01 11** (MIDI map `02 11`).

---

#### Global patch settings (Routing, Level, Tuning)

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **01 24** | 02 24 | **GK SET Select** | 0 - 10 | 0=SYSTEM, 1-10=SET 1-10 |
| **01 25** | 02 25 | **Guitar Out Source** | 0 - 3 | OFF, NORMAL, MODEL, BOTH |
| **01 26** | 02 26 | **V-LINK Palette** | 0 - 32 | LAST, 1 - 32 |
| **01 27** | 02 27 | **V-LINK Patch Clip** | 0 - 32 | LAST, 1 - 32 |
| **01 28** | 02 28 | **V-LINK Clip Change**| 0 - 4 | OFF, 1 - 4 |
| **01 29** | 02 29 | **V-LINK EXP Pedal** | 0 - 4 | OFF, Col, Brgt, Speed |
| **01 2A** | 02 2A | **V-LINK EXP On** | 0 - 4 | OFF, Col, Brgt, Speed |
| **01 2B** | 02 2B | **V-LINK GK VOL** | 0 - 4 | OFF, Col, Brgt, Speed |
| **01 2C** | 02 2C | **EFFECT Structure** | 0 - 1 | 0=Str 1, 1=Str 2 |
| **01 2D** | 02 2D | **Line Select Model** | 0 - 2 | BYPS, AMP, MFX |
| **01 2E** | 02 2E | **Line Sel Normal PU**| 0 - 2 | BYPS, AMP, MFX |
| **01 2F** | 02 2F | **(Reserved)** | 0 - 127 | Reserved |
| **01 30** | 02 30 | **Patch Level** | 0 - 200 | **Nibble (*2)** |
| **01 32** | 02 32 | **Normal PU Mute** | 0 - 1 | 0=OFF, 1=ON |
| **01 33** | 02 33 | **Normal PU Level** | 0 - 100 | Volume 0 - 100 |
| **01 34** | 02 34 | **Alt Tune Switch** | 0 - 1 | 0=OFF, 1=ON |
| **01 35** | 02 35 | **Alt Tune Type** | 0 - 12 | OPEN, DROP, STEP, USER, etc. |
| **01 36 - 01 3B** | 02 36 - 02 3B | **User Tune String 1-6**| 40 - 88 | -24 to +24 semitones |
| **01 3C** | 02 3C | **Patch Tempo** | 20 - 250 | **Nibble (*2)** BPM |
| **01 3E** | 02 3E | **BYPS Chorus Send** | 0 - 100 | Chorus send when bypassed |
| **01 3F** | 02 3F | **BYPS Delay Send** | 0 - 100 | Delay send when bypassed |
| **01 40** | 02 40 | **BYPS Reverb Send** | 0 - 100 | Reverb send when bypassed |
| **01 42** | 02 42 | **EXP(OFF) MOD Ctrl Min**| 0 - 120 | Mod Control Min |
| **01 43** | 02 43 | **EXP(OFF) MOD Ctrl Max**| 0 - 120 | Mod Control Max |
| **01 44** | 02 44 | **EXP(ON) MOD Ctrl Min** | 0 - 120 | Mod Control Min |
| **01 45** | 02 45 | **EXP(ON) MOD Ctrl Max** | 0 - 120 | Mod Control Max |
| **01 46** | 02 46 | **GK VOL MOD Ctrl Min** | 0 - 120 | Mod Control Min |
| **01 47** | 02 47 | **GK VOL MOD Ctrl Max** | 0 - 120 | Mod Control Max |

**Field types:**
*   **0aaa aaaa**: Standard 7-bit data.
*   **Nibble (*2)**: Value split across several bytes (usually 3 or 4 bits per byte) for ranges above 127.
*   ***C127 / *C64**: Internal conversion tables for how the SysEx value is shown on the unit display.

### 4.1 Chorus Parsing

Offsets under Temporary Patch CHO/DLY/REV/EQ (`18 00 06 00`). Address order: Chorus → Delay → Reverb → EQ.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **Chorus Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 01** | 00 01 | **Chorus Type** | 0 - 3 | `0000 00aa` · 0: MONO, 1: STEREO, 2: MONO MILD, 3: STEREO MILD |
| **00 02** | 00 02 | **Chorus Rate** | 0 - 113 | `0aaa aaaa` · 0 – 100, WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 03** | 00 03 | **Chorus Depth** | 0 - 100 | `0aaa aaaa` · 0 – 100 |
| **00 04** | 00 04 | **Chorus Effect Level** | 0 - 100 | `0aaa aaaa` · 0 – 100 |

---

### 4.2 Delay Parsing

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 05** | 00 05 | **Delay Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 06** | 00 06 | **Delay Type** | 0 - 6 | `0000 0aaa` · 0: SINGLE, 1: PAN, 2: REVERSE, 3: ANALOG, 4: TAPE, 5: MODULATE, 6: HICUT |
| **00 07** | 00 07 | **Delay Time** | 1 - 3413 | `0000 aaaa` · Triple Nibble Calculation Required (see logic below) |
| **00 08** | 00 08 | **(Part of Delay Time)** |  | `0000 bbbb` |
| **00 09** | 00 09 | **(Part of Delay Time)** |  | `0000 cccc` · Values: 1–3400ms, 16TH, 8TH3, 16THd, 8TH, 4TH3, 8THd, 4TH, 2TH3, 4THd, 2TH, WHL3, 2THd, WHL |
| **00 0A** | 00 0A | **Delay Feedback** | 0 - 100 | `0aaa aaaa` · 0 – 100 |
| **00 0B** | 00 0B | **Delay Effect Level** | 0 - 120 | `0aaa aaaa` · 0 – 120 |

---

### 4.3 Reverb Parsing

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 0C** | 00 0C | **Reverb Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 0D** | 00 0D | **Reverb Type** | 0 - 4 | `0000 0aaa` · 0: AMBIENCE, 1: ROOM, 2: HALL1, 3: HALL2, 4: PLATE |
| **00 0E** | 00 0E | **Reverb Time** | 0 - 99 | `0aaa aaaa` · 0.1 – 10.0 [sec] |
| **00 0F** | 00 0F | **Reverb High Cut** | 0 - 9 | `0000 aaaa` · 0: 700, 1: 1000, 2: 1400, 3: 2000, 4: 3000, 5: 4000, 6: 6000, 7: 8000, 8: 11000, 9: FLAT [Hz] |
| **00 10** | 00 10 | **Reverb Effect Level** | 0 - 100 | `0aaa aaaa` · 0 – 100 |

---

### 4.4 EQ Parsing

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 11** | 00 11 | **EQ Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 12** | 00 12 | **EQ Low Cutoff Freq** | 0 - 10 | `0000 aaaa` · 0: FLAT, 1: 55, 2: 110, 3: 165, 4: 200, 5: 280, 6: 340, 7: 400, 8: 500, 9: 630, 10: 800 [Hz] |
| **00 13** | 00 13 | **EQ Low Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 [dB] (Value 20 = 0dB) |
| **00 14** | 00 14 | **EQ Low Mid Cutoff** | 0 - 27 | `000a aaaa` · 0: 20, 1: 25, 2: 31.5, 3: 40, 4: 50, 5: 63, 6: 80, 7: 100, 8: 125, 9: 160, 10: 200, 11: 250, 12: 315, 13: 400, 14: 500, 15: 630, 16: 800, 17: 1000, 18: 1250, 19: 1600, 20: 2000, 21: 2500, 22: 3150, 23: 4000, 24: 5000, 25: 6300, 26: 8000, 27: 10000 [Hz] |
| **00 15** | 00 15 | **EQ Low Mid Q** | 0 - 5 | `0000 0aaa` · 0: 0.5, 1: 1, 2: 2, 3: 4, 4: 8, 5: 16 |
| **00 16** | 00 16 | **EQ Low Mid Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 [dB] |
| **00 17** | 00 17 | **EQ High Mid Cutoff** | 0 - 27 | `000a aaaa` · 20Hz – 10000Hz (Same mapping as Low Mid Cutoff) |
| **00 18** | 00 18 | **EQ High Mid Q** | 0 - 5 | `0000 0aaa` · 0: 0.5, 1: 1, 2: 2, 3: 4, 4: 8, 5: 16 |
| **00 19** | 00 19 | **EQ High Mid Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 [dB] |
| **00 1A** | 00 1A | **EQ High Cutoff Freq** | 0 - 9 | `0000 aaaa` · 0: 700, 1: 1000, 2: 1400, 3: 2000, 4: 3000, 5: 4000, 6: 6000, 7: 8000, 8: 11000, 9: FLAT [Hz] |
| **00 1B** | 00 1B | **EQ High Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 [dB] |
| **00 1C** | 00 1C | **EQ Level** | 0 - 40 | `00aa aaaa` · -20 – +20 [dB] |

---

### 4.5 Amp Parsing


| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **Amp Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 01** | 00 01 | **Amp Type** | 0 - 41 | `00aa aaaa` · 0: BOSS CLEAN, 1: JC-120, 2: JAZZ COMBO, 3: FULL RANGE, 4: CLEAN TWIN, 5: PRO CRUNCH, 6: TWEED, 7: DELUX CRUNCH, 8: BOSS CRUNCH, 9: BLUES, 10: WILD CRUNCH, 11: STACK CRUNCH, 12: VO DRIVE, 13: VO LEAD, 14: VO CLEAN, 15: MATCH DRIVE, 16: FAT MATCH, 17: MATCH LEAD, 18: BG LEAD, 19: BG DRIVE, 20: BG RHYTHM, 21: MS 1959 I, 22: MS 1959 I+II, 23: MS HIGAIN, 24: MS SCOOP, 25: R-FIER VINTAGE, 26: R-FIER MODERN, 27: R-FIER CLEAN, 28: T-AMP LEAD, 29: T-AMP CRUNCH, 30: T-AMP CLEAN, 31: BOSS DRIVE, 32: SLDN, 33: LEAD STACK, 34: HEAVY LEAD, 35: BOSS METAL, 36: 5150 DRIVE, 37: METAL LEAD, 38: EDGE LEAD, 39: BASS CLEAN, 40: BASS CRUNCH, 41: BASS HIGAIN |
| **00 02** | 00 02 | **Amp Gain** | 0 - 120 | `0aaa aaaa` · Absolute value (0 – 120) |
| **00 03** | 00 03 | **Amp Level** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 04** | 00 04 | **Amp Gain Switch** | 0 - 2 | `0000 00aa` · 0: LOW, 1: MID, 2: HIGH |
| **00 05** | 00 05 | **Amp Solo Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 06** | 00 06 | **Amp Solo Level** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 07** | 00 07 | **Amp Bass** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 08** | 00 08 | **Amp Middle** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 09** | 00 09 | **Amp Treble** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 0A** | 00 0A | **Amp Presence** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |
| **00 0B** | 00 0B | **Amp Bright** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 0C** | 00 0C | **Amp SP type** | 0 - 8 | `0000 aaaa` · 0: OFF, 1: ORIGIN, 2: 1x8", 3: 1x10", 4: 1x12", 5: 2x12", 6: 4x10", 7: 4x12", 8: 8x12 |
| **00 0D** | 00 0D | **Amp Mic Type** | 0 - 4 | `0000 0aaa` · 0: DYN57, 1: DYN421, 2: CND451, 3: CND87, 4: FLAT |
| **00 0E** | 00 0E | **Amp Mic Distance** | 0 - 1 | `0000 000a` · 0: OFF MIC, 1: ON MIC |
| **00 0F** | 00 0F | **Amp Mic Position** | 0 - 10 | `0000 aaaa` · 0: CENTER, 1 – 10 |
| **00 10** | 00 10 | **Amp Mic Level** | 0 - 100 | `0aaa aaaa` · Absolute value (0 – 100) |

---

### 4.6 MOD Parsing
To parse the **MOD Effect** parameters from the provided technical documentation, follow these precise instructions. The MOD section is unique as it contains multiple sub-types (OD/DS, Wah, etc.) that share the same memory space depending on the **MOD Type** selection.

#### 4.6.1 Core MOD Controls (Shared)
These parameters manage the general state and signal routing of the MOD block.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 11** | 00 11 | **MOD Chorus Send Level** | 0 - 100 | `0aaa aaaa` · Absolute Level |
| **00 12** | 00 12 | **MOD Delay Send Level** | 0 - 100 | `0aaa aaaa` · Absolute Level |
| **00 13** | 00 13 | **MOD Reverb Send Level** | 0 - 100 | `0aaa aaaa` · Absolute Level |
| **00 15** | 00 15 | **MOD Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 16** | 00 16 | **MOD Type** | 0 - 13 | `0000 aaaa` · 0: OD/DS, 1: WAH, 2: COMP, 3: LIMITER, 4: OCTAVE, 5: PHASER, 6: FLANGER, 7: TREMOLO, 8: ROTARY, 9: UNI-V, 10: PAN, 11: DELAY, 12: CHORUS, 13: EQ |
| **00 17** | 00 17 | **MOD Pan** | 0 - 100 | `0aaa aaaa` · L50 – 50R |

---

#### 4.6.2 OD/DS (Overdrive/Distortion) Sub-Parameters
Selected when **Offset 00 16** is set to `0`.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 18** | 00 18 | **ODDS Type** | 0 - 24 | `000a aaaa` · MID BOOST, CLEAN BOOST, TREBLE BST, BLUES OD, CRUNCH, NATURAL OD, OD-1, T-SCREAM, TURBO OD, WARM OD, DISTORTION, MILD DS, MID DS, RAT, GUV DS, DST+, MODERN DS, SOLID DS, STACK, LOUD, METAL ZONE, LEAD, '60S FUZZ, OCT FUZZ, MUFF FUZZ |
| **00 19** | 00 19 | **ODDS Drive** | 0 - 120 | `0aaa aaaa` · Absolute Value |
| **00 1A** | 00 1A | **ODDS Tone** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 1B** | 00 1B | **ODDS Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |

---

#### 4.6.3 Wah, Compressor, Limiter & Octave Sub-Parameters
Selected when **Offset 00 16** is set to `1, 2, 3,` or `4`.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 1C** | 00 1C | **WAH Mode** | 0 - 2 | `0000 00aa` · 0: MANUAL, 1: T.UP, 2: T.DOWN |
| **00 1D** | 00 1D | **WAH Type** | 0 - 5 | `0000 0aaa` · 0: CRY WAH, 1: VO WAH, 2: FAT WAH, 3: LIGHT WAH, 4: 7STRING WAH, 5: RESO WAH |
| **00 1E** | 00 1E | **WAH Pedal Position** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 1F** | 00 1F | **WAH Sens** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 20** | 00 20 | **WAH Freq** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 21** | 00 21 | **WAH Peak** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 22** | 00 22 | **WAH Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 23** | 00 23 | **COMP Sustain** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 24** | 00 24 | **COMP Attack** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 25** | 00 25 | **COMP Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 26** | 00 26 | **LIMITER Threshold** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 27** | 00 27 | **LIMITER Release** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 28** | 00 28 | **LIMITER Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 29** | 00 29 | **OCTAVE Oct Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 2A** | 00 2A | **OCTAVE Dry Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |

---

#### 4.6.4 Modulation Effects (Phaser, Flanger, Tremolo)
Selected when **Offset 00 16** is set to `5, 6,` or `7`. Note the shared use of the "Rate" mapping for musical notes.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 2B** | 00 2B | **PHASER Type** | 0 - 3 | `0000 00aa` · 4STAGE, 8STAGE, 12STAGE, BI-PHASE |
| **00 2C** | 00 2C | **PHASER Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values (2THd – 16TH) |
| **00 2D** | 00 2D | **PHASER Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 2E** | 00 2E | **PHASER Resonance** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 2F** | 00 2F | **PHASER Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 30** | 00 30 | **FLANGER Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 31** | 00 31 | **FLANGER Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 32** | 00 32 | **FLANGER Manual** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 33** | 00 33 | **FLANGER Resonance** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 34** | 00 34 | **FLANGER Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 35** | 00 35 | **TREMOLO Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 36** | 00 36 | **TREMOLO Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 37** | 00 37 | **TREMOLO Wave Shape** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 38** | 00 38 | **TREMOLO Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |

---

#### 4.6.5 Specialty Modulation (Rotary, Uni-V, Pan)
Selected when **Offset 00 16** is set to `8, 9,` or `10`.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 39** | 00 39 | **ROTARY Rate Slow** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 3A** | 00 3A | **ROTARY Rate Fast** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 3B** | 00 3B | **ROTARY Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 3C** | 00 3C | **ROTARY Select** | 0 - 1 | `0000 000a` · 0: SLOW, 1: FAST |
| **00 3D** | 00 3D | **ROTARY Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 3E** | 00 3E | **UNI-V Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 3F** | 00 3F | **UNI-V Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 40** | 00 40 | **UNI-V Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 41** | 00 41 | **PAN Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 42** | 00 42 | **PAN Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 43** | 00 43 | **PAN Wave Shape** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 44** | 00 44 | **PAN Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |

---

#### 4.6.6 MOD Delay & Chorus
Selected when **Offset 00 16** is set to `11` or `12`. Note the **Triple Nibble** logic for Delay Time.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 45** | 00 45 | **DELAY Type** | 0 - 7 | `0000 0aaa` · SINGLE, PAN, STEREO, REVERSE, ANALOG, TAPE, MODULATE, HICUT |
| **00 46** | 00 46 | **DELAY Time** | 1 - 3413 | `0000 aaaa` · **Triple Nibble** (Offsets 46, 47, 48) |
| **00 49** | 00 49 | **DELAY Feedback** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 4A** | 00 4A | **DELAY Effect Level** | 0 - 120 | `0aaa aaaa` · Absolute Value |
| **00 4B** | 00 4B | **CHORUS Type** | 0 - 5 | `0000 0aaa` · MONO, STEREO1, STEREO2, MONO MILD, STEREO1 MILD, STEREO2 MILD |
| **00 4C** | 00 4C | **CHORUS Rate** | 0 - 113 | `0aaa aaaa` · 0–100, then Note Values |
| **00 4D** | 00 4D | **CHORUS Depth** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 4E** | 00 4E | **CHORUS Effect Level** | 0 - 100 | `0aaa aaaa` · Absolute Value |

---

#### 4.6.7 MOD EQ

EQ is selected when **Offset 00 16** is `13`. The Noise Suppressor follows the EQ section.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 4F** | 00 4F | **EQ Low Cutoff Freq** | 0 - 10 | `0000 aaaa` · FLAT, 55, 110, 165, 200, 280, 340, 400, 500, 630, 800 [Hz] |
| **00 50** | 00 50 | **EQ Low Gain** | 0 - 40 | `00aa aaaa` · -20 to +20 [dB] |
| **00 51** | 00 51 | **EQ Low Mid Cutoff** | 0 - 27 | `000a aaaa` · 20Hz – 10000Hz (Step mapping required) |
| **00 52** | 00 52 | **EQ Low Mid Q** | 0 - 5 | `0000 0aaa` · 0.5, 1, 2, 4, 8, 16 |
| **00 53** | 00 53 | **EQ Low Mid Gain** | 0 - 40 | `00aa aaaa` · -20 to +20 [dB] |
| **00 54** | 00 54 | **EQ High Mid Cutoff** | 0 - 27 | `000a aaaa` · 20Hz – 10000Hz (Same mapping as Low Mid) |
| **00 55** | 00 55 | **EQ High Mid Q** | 0 - 5 | `0000 0aaa` · 0.5, 1, 2, 4, 8, 16 |
| **00 56** | 00 56 | **EQ High Mid Gain** | 0 - 40 | `00aa aaaa` · -20 to +20 [dB] |
| **00 57** | 00 57 | **EQ High Gain** | 0 - 40 | `00aa aaaa` · -20 to +20 [dB] |
| **00 58** | 00 58 | **EQ High Cutoff Freq** | 0 - 9 | `0000 aaaa` · 700 – 11000, FLAT [Hz] |
| **00 59** | 00 59 | **EQ Level** | 0 - 40 | `00aa aaaa` · -20 to +20 [dB] |

### 4.7 Noise Suppressor (NS)


| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 5A** | 00 5A | **NS Switch** | 0 - 1 | `0000 000a` · 0: OFF, 1: ON |
| **00 5B** | 00 5B | **NS Threshold** | 0 - 100 | `0aaa aaaa` · Absolute Value |
| **00 5C** | 00 5C | **NS Release Time** | 0 - 100 | `0aaa aaaa` · Absolute Value |

### 4.8 MFX Parsing

Exhaustive MFX parameter list by sub-type. Live SSOT: `Data/mfx/MFXCore.json`, `MFXIndex.json`, and `MFX*.json` via `MfxParametersCatalog`. Semantics: [MFX.md](./MFX.md).

**Addressing:** Rows below use **app dump byte indices** inside Temporary Patch MFX (`18 00 03 00`). Where the MIDI Implementation prints `01 xx`, the app index is `01 xx − 0x80` (same rule as `MfxFieldTypeHelpers.correctedByteIndex`). Those rows show `(MIDI map `01 xx`)` for traceability. Do not use the MIDI map value alone as a dump index.

Row shape matches Patch Common: **App byte index** | **MIDI map** | **Parameter** | **Stored (SysEx)** | **Presentation / notes** (bit pattern folded into notes).

#### 4.8.1 Core MFX Controls (Shared)

Core MFX Parameters (Offsets 00 00 – 00 06)

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **MFX Chorus Send Level** | 0 - 100 | `0aaa aaaa` |
| **00 01** | 00 01 | **MFX Delay Send Level** | 0 - 100 | `0aaa aaaa` |
| **00 02** | 00 02 | **MFX Reverb Send Level** | 0 - 100 | `0aaa aaaa` |
| **00 03** | 00 03 | **(Reserved)** | 0 - 100 | `0aaa aaaa` |
| **00 04** | 00 04 | **MFX Switch** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 05** | 00 05 | **MFX Type** | 0 - 19 | `000a aaaa` · EQ, SUPER FILTER, PHASER, STEP PHASER, RING MODULATOR, TREMOLO, AUTO PAN, SLICER, VK ROTARY, HEXA-CHORUS, SPACE-D, FLANGER, STEP FLANGER, GUITAR AMP SIM, COMPRESSOR, LIMITER, 3TAP PAN DELAY, TIME CTRL DELAY, LOFI COMPRESS, PITCH SHIFTER |
| **00 06** | 00 06 | **MFX Pan** | 0 - 100 | `0aaa aaaa` · L50 – 50R |

---

#### 4.8.2 MFX EQ 

**MFX Type 0: EQ (Equalizer)**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 07** | 00 07 | **EQ Low Freq** | 0 - 1 | `0000 000a` · 200, 400 [Hz] |
| **00 08** | 00 08 | **EQ Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 09** | 00 09 | **EQ Mid1 Freq** | 0 - 16 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000 [Hz] |
| **00 0A** | 00 0A | **EQ Mid1 Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 0B** | 00 0B | **EQ Mid1 Q** | 0 - 4 | `0000 0aaa` · 0.5, 1.0, 2.0, 4.0, 8.0 |
| **00 0C** | 00 0C | **EQ Mid2 Freq** | 0 - 16 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000 [Hz] |
| **00 0D** | 00 0D | **EQ Mid2 Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 0E** | 00 0E | **EQ Mid2 Q** | 0 - 4 | `0000 0aaa` · 0.5, 1.0, 2.0, 4.0, 8.0 |
| **00 0F** | 00 0F | **EQ High Freq** | 0 - 2 | `0000 00aa` · 2000, 4000, 8000 [Hz] |
| **00 10** | 00 10 | **EQ High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 11** | 00 11 | **EQ Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.3 MFX Filter

**MFX Type 1: SUPER FILTER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 12** | 00 12 | **SUPER FILTER Filter Type** | 0 - 3 | `0000 00aa` · LPF, BPF, HPF, NOTCH |
| **00 13** | 00 13 | **SUPER FILTER Filter Slope** | 0 - 2 | `0000 00aa` · -12, -24, -36 [dB] |
| **00 14** | 00 14 | **SUPER FILTER Filter Cutoff** | 0 - 127 | `0aaa aaaa` |
| **00 15** | 00 15 | **SUPER FILTER Filter Resonance** | 0 - 127 | `0aaa aaaa` |
| **00 16** | 00 16 | **SUPER FILTER Filter Gain** | 0 - 12 | `0000 aaaa` · 0 – +12 [dB] |
| **00 17** | 00 17 | **SUPER FILTER Modulation Sw** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 18** | 00 18 | **SUPER FILTER Modulation Wave** | 0 - 4 | `0000 0aaa` · TRI, SQR, SIN, SAW1, SAW2 |
| **00 19** | 00 19 | **SUPER FILTER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 1A** | 00 1A | **SUPER FILTER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 1B** | 00 1B | **SUPER FILTER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 1C** | 00 1C | **SUPER FILTER Depth** | 0 - 127 | `0aaa aaaa` |
| **00 1D** | 00 1D | **SUPER FILTER Attack** | 0 - 127 | `0aaa aaaa` |
| **00 1E** | 00 1E | **SUPER FILTER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.4 MFX Phaser

**MFX Type 2: PHASER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 1F** | 00 1F | **PHASER Mode** | 0 - 2 | `0000 00aa` · 4-STAGE, 8-STAGE, 12-STAGE |
| **00 20** | 00 20 | **PHASER Manual** | 0 - 127 | `0aaa aaaa` |
| **00 21** | 00 21 | **PHASER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 22** | 00 22 | **PHASER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 23** | 00 23 | **PHASER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 24** | 00 24 | **PHASER Depth** | 0 - 127 | `0aaa aaaa` |
| **00 25** | 00 25 | **PHASER Polarity** | 0 - 1 | `0000 000a` · INVERSE, SYNCHRO |
| **00 26** | 00 26 | **PHASER Resonance** | 0 - 127 | `0aaa aaaa` |
| **00 27** | 00 27 | **PHASER Cross Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 28** | 00 28 | **PHASER Mix** | 0 - 127 | `0aaa aaaa` |
| **00 29** | 00 29 | **PHASER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 2A** | 00 2A | **PHASER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 2B** | 00 2B | **PHASER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.5 MFX Step Phaser

**MFX Type 3: STEP PHASER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 2C** | 00 2C | **STEP PHASER Mode** | 0 - 2 | `0000 00aa` · 4-STAGE, 8-STAGE, 12-STAGE |
| **00 2D** | 00 2D | **STEP PHASER Manual** | 0 - 127 | `0aaa aaaa` |
| **00 2E** | 00 2E | **STEP PHASER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 2F** | 00 2F | **STEP PHASER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 30** | 00 30 | **STEP PHASER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 31** | 00 31 | **STEP PHASER Depth** | 0 - 127 | `0aaa aaaa` |
| **00 32** | 00 32 | **STEP PHASER Polarity** | 0 - 1 | `0000 000a` · INVERSE, SYNCHRO |
| **00 33** | 00 33 | **STEP PHASER Resonance** | 0 - 127 | `0aaa aaaa` |
| **00 34** | 00 34 | **STEP PHASER Cross Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 35** | 00 35 | **STEP PHASER Step Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 36** | 00 36 | **STEP PHASER Step Rate** | 0 - 100 | `0aaa aaaa` |
| **00 37** | 00 37 | **STEP PHASER Step Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 38** | 00 38 | **STEP PHASER Mix** | 0 - 127 | `0aaa aaaa` |
| **00 39** | 00 39 | **STEP PHASER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 3A** | 00 3A | **STEP PHASER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 3B** | 00 3B | **STEP PHASER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.6 MFX Ring Modulator

**MFX Type 4: RING MODULATOR**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 3C** | 00 3C | **RING MODULATOR Frequency** | 0 - 127 | `0aaa aaaa` |
| **00 3D** | 00 3D | **RING MODULATOR Sens** | 0 - 127 | `0aaa aaaa` |
| **00 3E** | 00 3E | **RING MODULATOR Polarity** | 0 - 1 | `0000 000a` · UP, DOWN |
| **00 3F** | 00 3F | **RING MODULATOR Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 40** | 00 40 | **RING MODULATOR High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 41** | 00 41 | **RING MODULATOR Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 42** | 00 42 | **RING MODULATOR Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.7 MFX Tremolo

**MFX Type 5: TREMOLO**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 43** | 00 43 | **TREMOLO Mod Wave** | 0 - 4 | `0000 0aaa` · TRI, SQR, SIN, SAW1, SAW2 |
| **00 44** | 00 44 | **TREMOLO Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 45** | 00 45 | **TREMOLO Rate** | 0 - 100 | `0aaa aaaa` |
| **00 46** | 00 46 | **TREMOLO Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 47** | 00 47 | **TREMOLO Depth** | 0 - 127 | `0aaa aaaa` |
| **00 48** | 00 48 | **TREMOLO Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 49** | 00 49 | **TREMOLO High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 4A** | 00 4A | **TREMOLO Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.8 MFX Auto Pan

**MFX Type 6: AUTO PAN**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 4B** | 00 4B | **AUTO PAN Mod Wave** | 0 - 4 | `0000 0aaa` · TRI, SQR, SIN, SAW1, SAW2 |
| **00 4C** | 00 4C | **AUTO PAN Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 4D** | 00 4D | **AUTO PAN Rate** | 0 - 100 | `0aaa aaaa` |
| **00 4E** | 00 4E | **AUTO PAN Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 4F** | 00 4F | **AUTO PAN Depth** | 0 - 127 | `0aaa aaaa` |
| **00 50** | 00 50 | **AUTO PAN Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 51** | 00 51 | **AUTO PAN High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 52** | 00 52 | **AUTO PAN Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.9 MFX Slicer

**MFX Type 7: SLICER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 53** | 00 53 | **SLICER Pattern** | 1 - 20 | `000a aaaa` |
| **00 54** | 00 54 | **SLICER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 55** | 00 55 | **SLICER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 56** | 00 56 | **SLICER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 57** | 00 57 | **SLICER Attack** | 0 - 127 | `0aaa aaaa` |
| **00 58** | 00 58 | **SLICER Input Sync Sw** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 59** | 00 59 | **SLICER Input Sync Threshold** | 0 - 127 | `0aaa aaaa` |
| **00 5A** | 00 5A | **SLICER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.10 MFX Rotary

**MFX Type 8: VK ROTARY**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 5B** | 00 5B | **VK ROTARY Speed** | 0 - 1 | `0000 000a` · SLOW, FAST |
| **00 5C** | 00 5C | **VK ROTARY Brake** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 5D** | 00 5D | **VK ROTARY Woofer Slow Speed** | 0 - 100 | `0aaa aaaa` |
| **00 5E** | 00 5E | **VK ROTARY Woofer Fast Speed** | 0 - 100 | `0aaa aaaa` |
| **00 5F** | 00 5F | **VK ROTARY Woofer Trans Up** | 0 - 127 | `0aaa aaaa` |
| **00 60** | 00 60 | **VK ROTARY Woofer Trans Down** | 0 - 127 | `0aaa aaaa` |
| **00 61** | 00 61 | **VK ROTARY Woofer Level** | 0 - 127 | `0aaa aaaa` |
| **00 62** | 00 62 | **VK ROTARY Tweeter Slow Speed** | 0 - 100 | `0aaa aaaa` |
| **00 63** | 00 63 | **VK ROTARY Tweeter Fast Speed** | 0 - 100 | `0aaa aaaa` |
| **00 64** | 00 64 | **VK ROTARY Tweeter Trans Up** | 0 - 127 | `0aaa aaaa` |
| **00 65** | 00 65 | **VK ROTARY Tweeter Trans Down** | 0 - 127 | `0aaa aaaa` |
| **00 66** | 00 66 | **VK ROTARY Tweeter Level** | 0 - 127 | `0aaa aaaa` |
| **00 67** | 00 67 | **VK ROTARY Spread** | 0 - 10 | `0000 aaaa` |
| **00 68** | 00 68 | **VK ROTARY Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 69** | 00 69 | **VK ROTARY High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 6A** | 00 6A | **VK ROTARY Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.11 MFX Chorus

**MFX Type 9: HEXA-CHORUS**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 6B** | 00 6B | **HEXA-CHORUS Pre Delay** | 0 - 125 | `0aaa aaaa` · 0.0 – 100 [msec] |
| **00 6C** | 00 6C | **HEXA-CHORUS Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 6D** | 00 6D | **HEXA-CHORUS Rate** | 0 - 100 | `0aaa aaaa` |
| **00 6E** | 00 6E | **HEXA-CHORUS Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 6F** | 00 6F | **HEXA-CHORUS Depth** | 0 - 127 | `0aaa aaaa` |
| **00 70** | 00 70 | **HEXA-CHORUS Pre Delay Deviation** | 0 - 20 | `000a aaaa` |
| **00 71** | 00 71 | **HEXA-CHORUS Depth Deviation** | 0 - 40 | `00aa aaaa` · -20 – +20 |
| **00 72** | 00 72 | **HEXA-CHORUS Pan Deviation** | 0 - 20 | `000a aaaa` |
| **00 73** | 00 73 | **HEXA-CHORUS Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 74** | 00 74 | **HEXA-CHORUS Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.12 MFX Space-D

**MFX Type 10: SPACE-D**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 75** | 00 75 | **SPACE-D Pre Delay** | 0 - 125 | `0aaa aaaa` · 0.0 – 100 [msec] |
| **00 76** | 00 76 | **SPACE-D Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 77** | 00 77 | **SPACE-D Rate** | 0 - 100 | `0aaa aaaa` |
| **00 78** | 00 78 | **SPACE-D Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 79** | 00 79 | **SPACE-D Depth** | 0 - 127 | `0aaa aaaa` |
| **00 7A** | 00 7A | **SPACE-D Phase** | 0 - 90 | `0aaa aaaa` · 0 – 180 [deg] |
| **00 7B** | 00 7B | **SPACE-D Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 7C** | 00 7C | **SPACE-D High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 7D** | 00 7D | **SPACE-D Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 7E** | 00 7E | **SPACE-D Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.13 MFX Flanger

**MFX Type 11: FLANGER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 7F** | 00 7F | **FLANGER Filter Type** | 0 - 2 | `0000 00aa` · OFF, LPF, HPF |
| **00 80** | 01 00 | **FLANGER Cutoff Freq** | 0 - 16 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000 [Hz] |
| **00 81** | 01 01 | **FLANGER Pre Delay** | 0 - 125 | `0aaa aaaa` · 0.0 – 100 [msec] |
| **00 82** | 01 02 | **FLANGER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 83** | 01 03 | **FLANGER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 84** | 01 04 | **FLANGER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 85** | 01 05 | **FLANGER Depth** | 0 - 127 | `0aaa aaaa` |
| **00 86** | 01 06 | **FLANGER Phase** | 0 - 90 | `0aaa aaaa` · 0 – 180 [deg] |
| **00 87** | 01 07 | **FLANGER Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 88** | 01 08 | **FLANGER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 89** | 01 09 | **FLANGER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 8A** | 01 0A | **FLANGER Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 8B** | 01 0B | **FLANGER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.14 MFX Step Flanger

**MFX Type 12: STEP FLANGER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 8C** | 01 0C | **STEP FLANGER Filter Type** | 0 - 2 | `0000 00aa` · OFF, LPF, HPF |
| **00 8D** | 01 0D | **STEP FLANGER Cutoff Freq** | 0 - 16 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000 [Hz] |
| **00 8E** | 01 0E | **STEP FLANGER Pre Delay** | 0 - 125 | `0aaa aaaa` · 0.0 – 100 [msec] |
| **00 8F** | 01 0F | **STEP FLANGER Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 90** | 01 10 | **STEP FLANGER Rate** | 0 - 100 | `0aaa aaaa` |
| **00 91** | 01 11 | **STEP FLANGER Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 92** | 01 12 | **STEP FLANGER Depth** | 0 - 127 | `0aaa aaaa` |
| **00 93** | 01 13 | **STEP FLANGER Phase** | 0 - 90 | `0aaa aaaa` · 0 – 180 [deg] |
| **00 94** | 01 14 | **STEP FLANGER Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 95** | 01 15 | **STEP FLANGER Step Rate (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 96** | 01 16 | **STEP FLANGER Step Rate** | 0 - 100 | `0aaa aaaa` |
| **00 97** | 01 17 | **STEP FLANGER Step Rate (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 98** | 01 18 | **STEP FLANGER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 99** | 01 19 | **STEP FLANGER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 9A** | 01 1A | **STEP FLANGER Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 9B** | 01 1B | **STEP FLANGER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.15 MFX Guitar Amp Sim

**MFX Type 13: GUITAR AMP SIM**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 9C** | 01 1C | **GTR AMP SIM Pre Amp Sw** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 9D** | 01 1D | **GTR AMP SIM Pre Amp Type** | 0 - 13 | `0000 aaaa` · JC-120, CLEAN TWIN, MATCH DRIVE, BG LEAD, MS1959 I, MS1959 II, MS1959 I+II, SLDN LEAD, METAL 5150, METAL LEAD, OD-1, OD-2 TURBO, DISTORTION, FUZZ |
| **00 9E** | 01 1E | **GTR AMP SIM Pre Amp Volume** | 0 - 127 | `0aaa aaaa` |
| **00 9F** | 01 1F | **GTR AMP SIM Pre Amp Master** | 0 - 127 | `0aaa aaaa` |
| **00 A0** | 01 20 | **GTR AMP SIM Pre Amp Gain** | 0 - 2 | `0000 00aa` · LOW, MIDDLE, HIGH |
| **00 A1** | 01 21 | **GTR AMP SIM Pre Amp Bass** | 0 - 127 | `0aaa aaaa` |
| **00 A2** | 01 22 | **GTR AMP SIM Pre Amp Middle** | 0 - 127 | `0aaa aaaa` |
| **00 A3** | 01 23 | **GTR AMP SIM Pre Amp Treble** | 0 - 127 | `0aaa aaaa` |
| **00 A4** | 01 24 | **GTR AMP SIM Pre Amp Presence (0 – 127)** | Match Drive -127 - 0 | `0aaa aaaa` |
| **00 A5** | 01 25 | **GTR AMP SIM Pre Amp Bright** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 A6** | 01 26 | **GTR AMP SIM Speaker Sw** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 A7** | 01 27 | **GTR AMP SIM Speaker Type** | 0 - 15 | `0000 aaaa` · SMALL 1, SMALL 2, MIDDLE, JC-120, BUILT-IN 1, BUILT-IN 2, BUILT-IN 3, BUILT-IN 4, BUILT-IN 5, BG STACK 1, BG STACK 2, MS STACK 1, MS STACK 2, METAL STACK, 2-STACK, 3-STACK |
| **00 A8** | 01 28 | **GTR AMP SIM Mic Setting** | 0 - 2 | `0000 00aa` · 1, 2, 3 |
| **00 A9** | 01 29 | **GTR AMP SIM Mic Level** | 0 - 127 | `0aaa aaaa` |
| **00 AA** | 01 2A | **GTR AMP SIM Direct Level** | 0 - 127 | `0aaa aaaa` |
| **00 AB** | 01 2B | **GTR AMP SIM Pan** | 0 - 127 | `0aaa aaaa` · L64 – 63R |
| **00 AC** | 01 2C | **GTR AMP SIM Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.16 MFX Compressor

**MFX Type 14: COMPRESSOR**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 AD** | 01 2D | **COMPRESSOR Attack** | 0 - 127 | `0aaa aaaa` |
| **00 AE** | 01 2E | **COMPRESSOR Threshold** | 0 - 127 | `0aaa aaaa` |
| **00 AF** | 01 2F | **COMPRESSOR Post Gain** | 0 - 18 | `000a aaaa` · 0 – +18 [dB] |
| **00 B0** | 01 30 | **COMPRESSOR Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 B1** | 01 31 | **COMPRESSOR High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 B2** | 01 32 | **COMPRESSOR Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.17 MFX Limiter

**MFX Type 15: LIMITER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 B3** | 01 33 | **LIMITER Release** | 0 - 127 | `0aaa aaaa` |
| **00 B4** | 01 34 | **LIMITER Threshold** | 0 - 127 | `0aaa aaaa` |
| **00 B5** | 01 35 | **LIMITER Ratio** | 0 - 3 | `0000 00aa` · 1.5:1, 2:1, 4:1, 100:1 |
| **00 B6** | 01 36 | **LIMITER Post Gain** | 0 - 18 | `000a aaaa` · 0 – +18 [dB] |
| **00 B7** | 01 37 | **LIMITER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 B8** | 01 38 | **LIMITER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 B9** | 01 39 | **LIMITER Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.18 MFX PAN Delay

**MFX Type 16: 3TAP PAN DELAY**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 BA** | 01 3A | **3TAP DELAY Delay Left (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 BB** | 01 3B | **3TAP DELAY Delay Left (msec)** | 1 - 2600 | `0000 aaaa` |
| **00 BC** | 01 3C | **(Cont.)** |  | `0000 bbbb` |
| **00 BD** | 01 3D | **(Cont.)** |  | `0000 cccc` |
| **00 BE** | 01 3E | **3TAP DELAY Delay Left (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 BF** | 01 3F | **3TAP DELAY Delay Right (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 C0** | 01 40 | **3TAP DELAY Delay Right (msec)** | 1 - 2600 | `0000 aaaa` |
| **00 C1** | 01 41 | **(Cont.)** |  | `0000 bbbb` |
| **00 C2** | 01 42 | **(Cont.)** |  | `0000 cccc` |
| **00 C3** | 01 43 | **3TAP DELAY Delay Right (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 C4** | 01 44 | **3TAP DELAY Delay Center (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 C5** | 01 45 | **3TAP DELAY Delay Center (msec)** | 1 - 2600 | `0000 aaaa` |
| **00 C6** | 01 46 | **(Cont.)** |  | `0000 bbbb` |
| **00 C7** | 01 47 | **(Cont.)** |  | `0000 cccc` |
| **00 C8** | 01 48 | **3TAP DELAY Delay Center (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 C9** | 01 49 | **3TAP DELAY Center Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 CA** | 01 4A | **3TAP DELAY HF Damp** | 0 - 17 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000, BYPASS [Hz] |
| **00 CB** | 01 4B | **3TAP DELAY Left Level** | 0 - 127 | `0aaa aaaa` |
| **00 CC** | 01 4C | **3TAP DELAY Right Level** | 0 - 127 | `0aaa aaaa` |
| **00 CD** | 01 4D | **3TAP DELAY Center Level** | 0 - 127 | `0aaa aaaa` |
| **00 CE** | 01 4E | **3TAP DELAY Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 CF** | 01 4F | **3TAP DELAY High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 D0** | 01 50 | **3TAP DELAY Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 D1** | 01 51 | **3TAP DELAY Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.19 MFX Time Ctrl Delay

**MFX Type 17: TIME CTRL DELAY**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 D2** | 01 52 | **TIME CTRL DELAY Delay Time (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 D3** | 01 53 | **TIME CTRL DELAY Delay Time (msec)** | 1 - 1300 | `0000 aaaa` |
| **00 D4** | 01 54 | **(Cont.)** |  | `0000 bbbb` |
| **00 D5** | 01 55 | **(Cont.)** |  | `0000 cccc` |
| **00 D6** | 01 56 | **TIME CTRL DELAY Delay Time (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 D7** | 01 57 | **TIME CTRL DELAY Acceleration** | 0 - 15 | `0000 aaaa` |
| **00 D8** | 01 58 | **TIME CTRL DELAY Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 D9** | 01 59 | **TIME CTRL DELAY HF Damp** | 0 - 17 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000, BYPASS [Hz] |
| **00 DA** | 01 5A | **TIME CTRL DELAY Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 DB** | 01 5B | **TIME CTRL DELAY High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 DC** | 01 5C | **TIME CTRL DELAY Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 DD** | 01 5D | **TIME CTRL DELAY Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.20 MFX LOFI COMPRESS

**MFX Type 18: LOFI COMPRESS**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 DE** | 01 5E | **LOFI COMPRESS Pre Filter Type** | 0 - 5 | `0000 0aaa` · 1, 2, 3, 4, 5, 6 |
| **00 DF** | 01 5F | **LOFI COMPRESS LoFi Type** | 0 - 8 | `0000 aaaa` · 1, 2, 3, 4, 5, 6, 7, 8, 9 |
| **00 E0** | 01 60 | **LOFI COMPRESS Post Filter Type** | 0 - 2 | `0000 00aa` · OFF, LPF, HPF |
| **00 E1** | 01 61 | **LOFI COMPRESS Post Filter Cutoff** | 0 - 16 | `000a aaaa` · 200, 250, 315, 400, 500, 630, 800, 1000, 1250, 1600, 2000, 2500, 3150, 4000, 5000, 6300, 8000, BYPASS [Hz] |
| **00 E2** | 01 62 | **LOFI COMPRESS Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 E3** | 01 63 | **LOFI COMPRESS High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 E4** | 01 64 | **LOFI COMPRESS Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 E5** | 01 65 | **LOFI COMPRESS Level** | 0 - 127 | `0aaa aaaa` |

---

#### 4.8.21 MFX Pitch Shifter

**MFX Type 19: PITCH SHIFTER**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 E6** | 01 66 | **PITCH SHIFTER Coarse** | 0 - 36 | `00aa aaaa` · -24 – +12 [semi] |
| **00 E7** | 01 67 | **PITCH SHIFTER Fine** | 0 - 100 | `0aaa aaaa` · -100 – +100 [cent] |
| **00 E8** | 01 68 | **PITCH SHIFTER Delay Time (sync sw)** | 0 - 1 | `0000 000a` · OFF, ON |
| **00 E9** | 01 69 | **PITCH SHIFTER Delay Time (msec)** | 1 - 1300 | `0000 aaaa` |
| **00 EA** | 01 6A | **(Cont.)** |  | `0000 bbbb` |
| **00 EB** | 01 6B | **(Cont.)** |  | `0000 cccc` |
| **00 EC** | 01 6C | **PITCH SHIFTER Delay Time (note)** | 0 - 12 | `0000 aaaa` · WHL, 2THd, WHL3, 2TH, 4THd, 2TH3, 4TH, 8THd, 4TH3, 8TH, 16THd, 8TH3, 16TH |
| **00 ED** | 01 6D | **PITCH SHIFTER Feedback** | 0 - 98 | `0aaa aaaa` · -98 – +98 [%] |
| **00 EE** | 01 6E | **PITCH SHIFTER Low Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 EF** | 01 6F | **PITCH SHIFTER High Gain** | 0 - 30 | `000a aaaa` · -15 – +15 [dB] |
| **00 F0** | 01 70 | **PITCH SHIFTER Balance** | 0 - 100 | `0aaa aaaa` · D100:0W – D0:100W |
| **00 F1** | 01 71 | **PITCH SHIFTER Level** | 0 - 127 | `0aaa aaaa` |

### 4.9 PCM Tone Parameters

#### PCM Tone (20 and 21)


| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **Tone Bank Select MSB** | 0 - 127 | Bank Select (CC# 0) |
| **00 01** | 00 01 | **Tone Bank Select LSB** | 0 - 127 | Bank Select (CC# 32) |
| **00 02** | 00 02 | **Tone Program Number** | 0 - 127 | Program Change (PC) |
| **00 03** | 00 03 | **Mute Switch** | 0 - 1 | 0=OFF, 1=MUTE |
| **00 04** | 00 04 | **Part Level** | 0 - 127 | Volume (CC# 7). Scaling *C127 |
| **00 05** | 00 05 | **Part Octave Shift** | 61 - 67 | Corresponds to -3 to +3 octaves |
| **00 06** | 00 06 | **Chromatic** | 0 - 1 | 0=OFF, 1=ON |
| **00 07** | 00 07 | **Legato Switch** | 0 - 1 | 0=OFF, 1=ON |
| **00 08** | 00 08 | **Nuance Switch** | 0 - 1 | 0=OFF, 1=ON |
| **00 09** | 00 09 | **Part Pan** | 0 - 127 | Panning (CC# 10). Scaling *C64pan (L64-63R) |
| **00 0A** | 00 0A | **Part Coarse Tune** | 40 - 88 | Coarse Tune (-24 to +24 semitones) |
| **00 0B** | 00 0B | **Part Fine Tune** | 14 - 114 | Fine Tune (-50 to +50 cents) |
| **00 0C** | 00 0C | **Part Portamento Switch** | 0 - 2 | 0=OFF, 1=ON, 2=TONE |
| **00 0D - 00 0E** | 00 0D - 00 0E | **Portamento Time** | 0 - 127 | Nibble (*2). Scaling *C127 |
| **00 0F** | 00 0F | **Release Mode** | 0 - 1 | 1 or 2 |
| **00 10** | 00 10 | **String 1 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 11** | 00 11 | **String 2 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 12** | 00 12 | **String 3 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 13** | 00 13 | **String 4 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 14** | 00 14 | **String 5 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 15** | 00 15 | **String 6 Level** | 0 - 127 | Individual string volume (*C127) |
| **00 16** | 00 16 | **Part Output MFX Select** | 0 - 2 | 0=BYPS, 1=AMP, 2=MFX |

#### PCM Tone Offset (30 and 31)


| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **TVF Filter Type** | 0 - 7 | OFF, LPF, BPF, HPF, PKG, LPF2, LPF3, TONE |
| **00 01** | 00 01 | **TVF Cutoff Freq Offset** | 1 - 127 | *C63 (-63 to +63) |
| **00 02** | 00 02 | **TVF Resonance Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 03** | 00 03 | **TVF Cutoff Velo Sens** | 0 - 127 | *C64 (-64 to +63) |
| **00 04** | 00 04 | **TVF Cutoff Velo Curve** | 0 - 8 | FIXED, 1 - 7, TONE |
| **00 05** | 00 05 | **TVF Cutoff Keyfollow** | 44 - 84 | *C200 (-200 to +200) |
| **00 06** | 00 06 | **Nuance Cutoff Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 07** | 00 07 | **TVF Env Depth Offset** | 1 - 127 | *C63 (-63 to +63) |
| **00 08** | 00 08 | **TVF Env Time 1 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 09** | 00 09 | **TVF Env Time 2 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 0A** | 00 0A | **TVF Env Level 3 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 0B** | 00 0B | **TVF Env Time 4 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 0C** | 00 0C | **TVF Env T1 Velo Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 0D** | 00 0D | **TVF Env T1 Nuance Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 0E** | 00 0E | **TVA Level Velo Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 0F** | 00 0F | **TVA Level Velo Curve** | 0 - 8 | FIXED, 1 - 7, TONE |
| **00 10** | 00 10 | **TVA Env Time 1 Offset** | 0 - 127 | *C64 (-63 to +63) |
| **00 11** | 00 11 | **TVA Env Time 2 Offset** | 0 - 127 | *C64 (-63 to +63) |
| **00 12** | 00 12 | **TVA Env Level 3 Offset** | 0 - 127 | *C64 (-63 to +63) |
| **00 13** | 00 13 | **TVA Env Time 4 Offset** | 0 - 127 | *C64 (-63 to +63) |
| **00 14** | 00 14 | **TVA Env T1 Velo Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 15** | 00 15 | **TVA Env T1 Nuance Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 16** | 00 16 | **Nuance Level Sens** | 1 - 127 | *C63 (-63 to +63) |
| **00 17** | 00 17 | **Pitch Env Velo Sens** | 0 - 127 | *C64 (-64 to +63) |
| **00 18** | 00 18 | **Pitch Env Offset** | 52 - 76 | -12 to +12 semitones |
| **00 19** | 00 19 | **Pitch Env Time 1 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 1A** | 00 1A | **Pitch Env Time 2 Offset** | 0 - 127 | *C64 (-64 to +63) |
| **00 1B** | 00 1B | **Part Portamento Type** | 0 - 1 | 0=RATE, 1=TIME |
| **00 1C - 00 1D** | 00 1C - 00 1D | **LFO1 Rate** | 0 - 150 | Nibble (*2). 0-127, Musical Notes, Tone |
| **00 1E** | 00 1E | **LFO1 Pitch Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 1F** | 00 1F | **LFO1 TVF Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 20** | 00 20 | **LFO1 TVA Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 21** | 00 21 | **LFO1 Pan Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 22 - 00 23** | 00 22 - 00 23 | **LFO2 Rate** | 0 - 150 | Nibble (*2). 0-127, Musical Notes, Tone |
| **00 24** | 00 24 | **LFO2 Pitch Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 25** | 00 25 | **LFO2 TVF Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 26** | 00 26 | **LFO2 TVA Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |
| **00 27** | 00 27 | **LFO2 Pan Depth Offset** | 0 - 127 | *C63Off (OFF, -63 to +63) |

### 4.10 COSM / Modeling
Complete **Patch Modeling Tone** (COSM) parameter list from `gr55_MIDI.pdf` and the owner’s manual. Live JSON: `Patch_Modeling_Common.json`, `Patch_Modeling_Bass.json`. Semantics: [Tone_Types.md](./Tone_Types.md).

#### Addressing (Modeling Tone)
COSM / Modeling Tone lives in the Temporary Patch at base **`18 00 10 00`**. Block size is `00 00 01 56` bytes.

App byte index compensates the MIDI map gap (`≥ 0x0100` → subtract `0x0080`).

#### Patch Modeling Tone parameters (base `18 00 10 00`)

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00** | 00 00 | **Tone Category (GTR)** | 0 - 3 | 0=E.GTR, 1=AC, 2=E.BASS, 3=SYNTH |
| **00 01** | 00 01 | **Tone No: E.GTR (GTR)** | 0 - 9 | CLA-ST, MOD-ST, TE, LP, etc. |
| **00 02** | 00 02 | **Tone No: AC (GTR)** | 0 - 4 | STEEL, NYLON, SITAR, BANJO, RESO |
| **00 03** | 00 03 | **Tone No: E.BASS (GTR)**| 0 - 1 | JB, PB |
| **00 04** | 00 04 | **Tone No: SYNTH (GTR)** | 0 - 5 | ANALOG GR to BRASS |
| **00 05** | 00 05 | **Tone Category (BASS)** | 0 - 2 | 0=E.BASS, 1=SYNTH, 2=E.GTR |
| **00 06** | 00 06 | **Tone No: E.BASS (BASS)**| 0 - 8 | VINT JB, JB, PB, M-MAN, VIOLIN etc. |
| **00 07** | 00 07 | **Tone No: E.GTR (BASS)** | 0 - 1 | ST, LP |
| **00 08** | 00 08 | **Tone No: SYNTH (BASS)** | 0 - 5 | ANALOG GR to BRASS |
| **00 09** | 00 09 | **Level** | 0 - 100 | Modeling tone level |
| **00 0A** | 00 0A | **Mute Switch** | 0 - 1 | 0=ON (sound), 1=OFF (mute) |
| **00 0B - 00 10** | 00 0B - 00 10 | **String 1-6 Level** | 0 - 100 | Per-string volume |
| **00 11** | 00 11 | **Pitch Shift String 1** | 0 - 48 | -24 - +24 semitoner (Center 24) |
| **00 12** | 00 12 | **Pitch Fine String 1** | 0 - 100 | -50 - +50 cents (Center 50) |
| **00 1D** | 00 1D | **12-STR Switch** | 0 - 1 | 0=OFF, 1=ON |
| **00 1E** | 00 1E | **12-STR Direct Level** | 0 - 100 | Main-string volume |
| **00 1F** | 00 1F | **12-STR Shift Str 1** | 0 - 48 | -24 - +24 semitoner |
| **00 20** | 00 20 | **12-STR Fine Str 1** | 0 - 100 | -50 - +50 cents |
| **00 2B** | 00 2B | **NS Switch** | 0 - 1 | Noise Suppressor OFF/ON |
| **00 2C** | 00 2C | **NS Threshold** | 0 - 100 | Noise suppressor threshold |
| **00 2D** | 00 2D | **NS Release** | 0 - 100 | Noise suppressor release time |
| **00 31** | 00 31 | **E.GTR Volume** | 0 - 100 | Volume for guitar models |
| **00 32** | 00 32 | **E.GTR Tone** | 0 - 100 | Tone control (0=soft, 100=standard) |
| **00 33** | 00 33 | **AC Steel Type** | 0 - 4 | MA28, TRP-0, GB45, GB SML, GLD40 |
| **00 34** | 00 34 | **AC Steel Body** | 0 - 100 | Body resonance |
| **00 4D** | 00 4D | **GR-300 Cutoff** | 0 - 100 | VCF filter cutoff frequency |
| **00 4E** | 00 4E | **GR-300 Resonance** | 0 - 100 | VCF filter resonance |
| **00 4F** | 00 4F | **GR-300 Env Mod SW** | 0 - 2 | 0=OFF, 1=ON (Wah), 2=INV (Reverse) |
| **00 50** | 00 50 | **GR-300 Env Mod Sens** | 0 - 100 | Envelope mod sensitivity |
| **00 53** | 00 53 | **GR-300 Pitch A** | 0 - 24 | -12 - +12 semitoner |
| **00 5E** | 00 5E | **Wave Synth Type** | 0 - 1 | 0=SAW, 1=SQUARE |
| **00 60** | 00 60 | **Filter Bass Cutoff** | 0 - 100 | Filter cutoff (0=soft, 100=hard) |
| **00 6F** | 00 6F | **Brass Cutoff** | 0 - 100 | Filter cutoff for Brass model |
| **00 90** | 01 10 | **E.GTR PU Sel (BASS)** | 0 - 2 | Pickup select for guitar in Bass mode |
| **00 92** | 01 12 | **E.GTR Vol (BASS)** | 0 - 100 | Volume for guitar in Bass mode |
| **00 96** | 01 16 | **E.BASS Vol (BASS)** | 0 - 100 | Total volume for Bass models |
| **00 98** | 01 18 | **E.BASS Treble (BASS)** | 0 - 100 | Treble (*C64: -50 - +50) |
| **00 A2** | 01 22 | **GR-300 Mode (BASS)** | 0 - 2 | 0=VCO, 1=V+D, 2=DIST |
| **00 C6** | 01 46 | **Brass Cutoff (BASS)** | 0 - 100 | Filter cutoff for Brass (Bass mode) |

#### Technical notes (COSM / Modeling):
*   **Reversed Logic:** For **Modeling Tone Switch** (offset `00 0A`): `00` = tone ON, `01` = tone OFF.
*   **Pitch Shift:** Coarse pitch parameters (offset `00 11` etc.) use center 24 (`18` hex) = 0 semitones. Stored 0–48 maps to **-24 to +24 semitones**.
*   ***C64 scale:** Many tone controls (e.g. Treble/Bass on bass models) send 0–127 with 64 as zero; the display shows **-50 to +50**.
*   **12-String Model:** When 12-string is on, expression-pedal PITCH BEND cannot be used on the modeling tone.
*   **Hexa-VCO:** **ANALOG GR** (GR-300) models hexa-distortion with six independent sawtooths (one per string), so pitch shift can be applied per string.

---

**Note:** App indices above compensate the MIDI map jump at `00 7F`→`01 00`. Bass-mode parameters (MIDI map `01 10` and up) use MIDI − `0x0080`.

## 5. System Parameter Reference

System data in the GR-55 is organized by **Sub-Address** (the 3rd byte of the Address).
Base Address: `02 00 00 00`

### 5.1 Sub-Address Routing Map

| Sub-Address (Hex) | Block Name | Description |
|-------------------|------------|-------------|
| **00** | System Common | Global settings (Output, MIDI, Tuner). |
| **02** | System CTL/Assign | Pedal functions (CTL, EXP, GK Vol/Switches). |
| **04 - 0D** | GK Setup 1-10 | GK Pickup settings (User Sets 1-10). |

**Rx BANK/PC Maps** (PRG mode) are *not* keyed by the 3rd address byte alone. Absolute addresses:

| Absolute | Block |
|----------|--------|
| `02 01 00 00` | Guitar Rx PC MAP bank 0 (banks 1…63 stride `04 00`; bank 63 at `02 02 7C 00`) |
| `02 03 00 00` | Bass Rx PC MAP bank 0 (… bank 63 at `02 04 7C 00`) |

Per bank: PC1…PC128 × USplit12 (3 nibbles), wire 0–600 (`OFF, 1–600`). App index `(pc−1)×3` (384 bytes). MIDI map jumps `00 7F`→`01 01` (and page 2) are ignored for dumps — same address-gap policy as elsewhere.

RQ1 size for one bank is MIDI-map **`00 00 03 00`** (device returns 384 data bytes). Linear size `00 00 01 80` is not answered. Single-PC probe: size `00 00 00 03`.

**MAP SELECT (FIX/PRG)** only changes how the unit responds to **incoming** Bank Select MSB (CC#0, 0–63) + PC. It does **not** change SysEx dumps of map RAM. FIX LCD list ≠ PRG map contents.

---

### 5.2 System Common (Sub-Address `00`)

**Base Address**: `02 00 00 00`
**Offsets** are relative to this base.

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00** | 00 | **GK SET Select** | 0 - 9 (GK Set 1 - 10) | `Value` |
| **01** | 01 | **OUTPUT Select** | 0=LINE/PHONES, 1=JC-120, 2=SMALL, 3=COMBO, 4=STACK, 5=JC-120 RET, 6=COMBO RET, 7=STACK RET, 8=B-AMP Tweet, 9=B-AMP NoTweet | `Option` |
| **02** | 02 | **Assign Hold** | 0=OFF, 1=ON | `Bool` |
| **03** | 03 | **Patch Ctrl Ch** | 0 - 15 (Ch 1 - 16) | `Value` |
| **04** | 04 | **RX Switch** | 0=OFF, 1=ON | `Bool` |
| **05** | 05 | **TX Switch** | 0=OFF, 1=ON | `Bool` |
| **06** | 06 | **V-LINK MIDI Ch** | 0 - 15 (Ch 1 - 16) | `Value` |
| **07** | 07 | **Gtr2Midi Sw** | 0=OFF, 1=ON | `Bool` |
| **08** | 08 | **Gtr2Midi Mode** | 0=MONO, 1=POLY | `Option` |
| **09** | 09 | **Gtr2Midi Chromat** | 0=OFF, 1=ON | `Bool` |
| **0A** | 0A | **Gtr2Midi StringCh** | 0=1 - 6, 1=2 - 7, ... 10=11 - 16 | `Option` |
| **0B** | 0B | **Gtr2Midi DataThin** | 0=OFF, 1=ON | `Bool` |
| **0C** | 0C | **Gtr2Midi CTL CC#** | 0 - 63 (OFF, 1 - 31, 64 - 95) | `Value` |
| **0D** | 0D | **Gtr2Midi EXP CC#** | 0 - 63 | `Value` |
| **0E** | 0E | **Gtr2Midi BendRng** | 0 - 48 (-24 to +24) | `Value` |
| **0F** | 0F | **Gtr2Midi GKVOL CC** | 0 - 63 | `Value` |
| **10** | 10 | **Gtr2Midi GKS1 CC** | 0 - 63 | `Value` |
| **11** | 11 | **Gtr2Midi GKS2 CC** | 0 - 63 | `Value` |
| **12** | 12 | **RX MAP Select** | 0=FIX, 1=PRG | `Option` |
| **15** | 15 | **USB Dir Monitor** | 0=OFF, 1=ON | `Bool` |
| **16** | 16 | **GtrOut Source** | 0=PATCH, 1=OFF, 2=NRM PU, 3=MODEL, 4=BOTH | `Option` |
| **17** | 17 | **Tuner Pitch** | 0 - 10 (435 - 445 Hz) | `Value` |
| **18** | 18 | **Tuner Mute Sw** | 0=OFF, 1=ON | `Bool` |
| **1A** | 1A | **GUITAR/BASS** | 0=GUITAR, 1=BASS (Critical: Affects GK Setup) | `Option` |
| **1C** | 1C | **Audio Player Lvl** | 0 - 200 | `Value` |
| **1E** | 1E | **USB Audio In Lvl** | 0 - 200 | `Value` |
| **20** | 20 | **USB Audio Out Lvl** | 0 - 200 | `Value` |

---

### 5.3 System Control/Assign (Sub-Address `02`)

**Base Address**: `02 00 02 00`
This block contains settings for all physical controllers.

#### **CTL Pedal (Offsets 00-0C)**
| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00** | 00 | **Function** | 0=OFF, 1=PATCH, 2=HOLD, 3=TAP, 4=TONE SW, 5=AMP SW, 6=MOD SW, 7=MFX SW, 8=DLY SW, 9=REV SW, 10=CHO SW, ... (See System_Functions.json) |  |
| **01** | 01 | **Hold Type** | 0 - 3 (1 - 4) |  |
| **02** | 02 | **Hold Sw Mode** | 0=LATCH, 1=MOMENT |  |
| **03** | 03 | **Hold PCM1** | 0=OFF, 1=ON |  |
| **04** | 04 | **Hold PCM2** | 0=OFF, 1=ON |  |
| **05** | 05 | **OFF PCM1 Sw** | 0=OFF, 1=ON |  |
| **06** | 06 | **OFF PCM2 Sw** | 0=OFF, 1=ON |  |
| **07** | 07 | **OFF Model Sw** | 0=OFF, 1=ON |  |
| **08** | 08 | **OFF NrmPu Sw** | 0=OFF, 1=ON |  |
| **09** | 09 | **ON PCM1 Sw** | 0=OFF, 1=ON |  |
| **0A** | 0A | **ON PCM2 Sw** | 0=OFF, 1=ON |  |
| **0B** | 0B | **ON Model Sw** | 0=OFF, 1=ON |  |
| **0C** | 0C | **ON NrmPu Sw** | 0=OFF, 1=ON |  |

#### **Expression Pedal OFF (Offsets 0D-23, 79-7A)**
| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **0D** | 0D | **Function** |  | 0=OFF, 1=PATCH, 2=PATCH VOL, 3=TONE VOL, 4=BEND, 5=MOD, ... |
| **0E-11** | 0E-11 | **Vol Sw (P1,P2,M,N)** |  | 4 Bytes (PCM1, PCM2, Model, Normal) |
| **12** | 12 | **Bend Range** |  | 0-24 (-12 to +12) |
| **13-15** | 13-15 | **Bend Sw (P1,P2,M)** |  | 3 Bytes |
| **16** | 16 | **Mod Min** |  | 0-127 |
| **17** | 17 | **Mod Max** |  | 0-127 |
| **18-19** | 18-19 | **Mod Sw (P1,P2)** |  | 2 Bytes |
| **1A-1D** | 1A-1D | **XFade Pol (P1,P2,M,N)** |  | 0=OFF, 1=TOE, 2=HEEL |
| **1E-1F** | 1E-1F | **Delay Lvl Min/Max** |  | 0-127 |
| **20-21** | 20-21 | **Reverb Lvl Min/Max** |  | 0-127 |
| **22-23** | 22-23 | **Chorus Lvl Min/Max** |  | 0-127 |
| **...** | ... | **...** |  | ... |
| **79** | 79 | **Mod Control Min** |  | 0-127 |
| **7A** | 7A | **Mod Control Max** |  | 0-127 |

#### **Expression Pedal ON (Offsets 24-3A, 7B-7C)**
Structure is identical to "Expression Pedal OFF" but offset by +0x17 (23 bytes).
- **24**: Function
- **25-28**: Vol Sw
- ...
- **7B**: Mod Control Min
- **7C**: Mod Control Max

#### **Expression Switch (Offsets 3B-47)**
| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **3B** | 3B | **Function** |  | 0=OFF, 1=PATCH, 2=TAP, 3=TONE SW, ... |
| **3C** | 3C | **Mode** |  | 0=LATCH, 1=MOMENT |
| **3D-40** | 3D-40 | **OFF Sw (P1,P2,M,N)** |  | 4 Bytes |
| **41-44** | 41-44 | **ON Sw (P1,P2,M,N)** |  | 4 Bytes |
| **45** | 45 | **Loop Rec/Play Lvl** |  | 0-100 |
| **46** | 46 | **Loop Dub Lvl** |  | 0-100 |
| **47** | 47 | **Loop Stop Lvl** |  | 0-100 |

#### **GK Volume (Offsets 48-5E, 7D-7E)**
Structure is similar to Expression Pedal.
- **48**: Function
- **49-4C**: Vol Sw
- **4D**: Bend Range
- ...
- **7D**: Mod Control Min
- **7E**: Mod Control Max

#### **GK S1 / S2 (Offsets 5F-6B, 6C-78)**
Structure is identical to Expression Switch (Function, Mode, Off/On Switches...).
- **5F**: GK S1 Function
- **6C**: GK S2 Function

---

### 5.4 GK Setup (Sub-Address `04` - `0D`)
This block's structure changes based on the **GUITAR/BASS Select** parameter (Offset `0x1A` in System Common).

**GK Set Index Calculation**: `Index = SubAddress - 0x04` (e.g., `04` -> Index 0).

#### **Guitar Mode Layout**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 00 - 07** | 00 00 - 07 | **GK Set Name (1–8)** | 32 - 127 | `0aaa aaaa` · ASCII |
| **00 08** | 00 08 | **GK PU Type** | 0 - 6 | `0000 0aaa` · GK-3, GK-2A, PIEZO, PIEZO F, PIEZO G, PIEZO L, PIEZO R |
| **00 09 - 0A** | 00 09 - 0A | **Scale** | 0 - 162 | `0000 aaaa / 0000 bbbb` · 500 – 660, ST, LP |
| **00 0B** | 00 0B | **GK PU Phase** | 0 - 1 | `0000 000a` · NORMAL, INVERSE |
| **00 0C** | 00 0C | **GK PU Direction** | 0 - 1 | `0000 000a` · NORMAL, REVERSE |
| **00 0D** | 00 0D | **S1/S2 Position** | 0 - 1 | `0000 000a` · NORMAL, REVERSE |
| **00 0E** | 00 0E | **Normal PU Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 |
| **00 0F** | 00 0F | **Piezo Low** | 0 - 20 | `000a aaaa` · -10 – +10 |
| **00 10** | 00 10 | **Piezo High** | 0 - 20 | `000a aaaa` · -10 – +10 |
| **00 11** | 00 11 | **GK PU <-> BRIDGE 1** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 12** | 00 12 | **GK PU <-> BRIDGE 2** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 13** | 00 13 | **GK PU <-> BRIDGE 3** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 14** | 00 14 | **GK PU <-> BRIDGE 4** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 15** | 00 15 | **GK PU <-> BRIDGE 5** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 16** | 00 16 | **GK PU <-> BRIDGE 6** | 0 - 40 | `00aa aaaa` · 10.0 – 30.0 mm |
| **00 17** | 00 17 | **GK PU Sens 1** | 0 - 100 | `0aaa aaaa` |
| **00 18** | 00 18 | **GK PU Sens 2** | 0 - 100 | `0aaa aaaa` |
| **00 19** | 00 19 | **GK PU Sens 3** | 0 - 100 | `0aaa aaaa` |
| **00 1A** | 00 1A | **GK PU Sens 4** | 0 - 100 | `0aaa aaaa` |
| **00 1B** | 00 1B | **GK PU Sens 5** | 0 - 100 | `0aaa aaaa` |
| **00 1C** | 00 1C | **GK PU Sens 6** | 0 - 100 | `0aaa aaaa` |
| **00 1D** | 00 1D | **Velocity Dynamics** | 0 - 4 | `0000 0aaa` · 1 – 5 |
| **00 1E** | 00 1E | **Velocity Low Cut** | 0 - 10 | `0000 aaaa` |
| **00 1F** | 00 1F | **PCM Velocity Sens** | 1 - 10 | `00aa aaaa` |
| **00 20** | 00 20 | **Nuance Dynamics** | 0 - 10 | `0000 aaaa` |
| **00 21** | 00 21 | **Nuance Trim** | 0 - 10 | `0000 aaaa` |
| **00 22** | 00 22 | **Down Tuning** | 0 - 5 | `0000 0aaa` · 0 – -5 |

#### **Bass Mode Layout**

| App byte index | MIDI map | Parameter | Stored (SysEx) | Presentation / notes |
| :--- | :--- | :--- | :--- | :--- |
| **00 23 - 2A** | 00 23 - 2A | **GK Set Name (1–8)** | 32 - 127 | `0aaa aaaa` · ASCII |
| **00 2B** | 00 2B | **GK PU Type** | 0 - 4 | `0000 0aaa` · GK-3B, GK-2B, PIEZO, PIEZO G, PIEZO R |
| **00 2C - 2D** | 00 2C - 2D | **Scale** | 0 - 234 | `0000 aaaa / 0000 bbbb` · 710 – 940, SHORT, MEDIUM, LONG, EXT LONG |
| **00 2E** | 00 2E | **GK PU Position** | 0 - 7 | `0000 0aaa` · 4STR-1, 4STR-2, 4STR-3, 5STR-LO1, 5STR-LO2, 5STR-HI1, 5STR-HI2, 6STR |
| **00 2F** | 00 2F | **GK PU Phase** | 0 - 1 | `0000 000a` · NORMAL, INVERSE |
| **00 30** | 00 30 | **GK PU Direction** | 0 - 1 | `0000 000a` · NORMAL, REVERSE |
| **00 31** | 00 31 | **S1/S2 Position** | 0 - 1 | `0000 000a` · NORMAL, REVERSE |
| **00 32** | 00 32 | **Normal PU Gain** | 0 - 40 | `00aa aaaa` · -20 – +20 |
| **00 33** | 00 33 | **Piezo Low** | 0 - 20 | `000a aaaa` · -10 – +10 |
| **00 34** | 00 34 | **Piezo High** | 0 - 20 | `000a aaaa` · -10 – +10 |
| **00 35 - 3A** | 00 35 - 3A | **GK PU <-> BRIDGE (1–6)** | 0 - 100 | `0aaa aaaa` · 5.5 – 55.5 mm |
| **00 3B - 40** | 00 3B - 40 | **GK PU Sens (1–6)** | 0 - 100 | `0aaa aaaa` |
| **00 41** | 00 41 | **Velocity Dynamics** | 0 - 4 | `0000 0aaa` · 1 – 5 |
| **00 42** | 00 42 | **Velocity Low Cut** | 0 - 10 | `0000 aaaa` |
| **00 43** | 00 43 | **PCM Velocity Sens** | 1 - 10 | `0aaa aaaa` |
| **00 44** | 00 44 | **Nuance Dynamics** | 0 - 10 | `0000 aaaa` |
| **00 45** | 00 45 | **Nuance Trim** | 0 - 10 | `0000 aaaa` |
| **00 46** | 00 46 | **Down Tuning** | 0 - 5 | `0000 0aaa` · 0 – -5 |

## 6. Setup Data
To retrieve the ID of the patch currently loaded in the GR-55, you must access the **Setup** section of the system memory and read the specific MIDI Bank Select and Program Change values.

Based on the provided System Exclusive address maps and technical charts, retrieving the currently loaded patch ID requires a specific two-step process that accounts for whether the GR-55 is in **Guitar mode** or **Bass mode**.

### 6.1 Retrieve the Raw Memory Values

To find the active patch, you must read from the **Setup** area of the system memory. The start address for the Setup section is **01 00 00 00**.

You need to retrieve the bytes at the following two offsets within that address:

* **Offset 00 00:** Patch Bank Select MSB (BS MSB).
* **Offset 00 01:** Patch Program Change (PC).

### 6.2 Decode the Patch ID by Mode

Once you have the **BS MSB** and **PC** values, you must identify the patch ID by cross-referencing the correct table for your current instrument mode.

#### 6.2.1 **Guitar Mode Decoding Table**

| Category | Patch Range | BS MSB (Hex) | PC Range (Decimal) |
| --- | --- | --- | --- |
| **LEAD** | 01-1 to 40-3 | 16 (10H) | 0 – 119 |
| **RHYTHM** | 01-1 to 03-2 | 16 (10H) | 120 – 127 |
|  | 03-3 to 40-3 | 17 (11H) | 0 – 111 |
| **OTHER** | 01-1 to 06-1 | 17 (11H) | 112 – 127 |
|  | 06-2 to 40-3 | 18 (12H) | 0 – 103 |
| **USER** | 01-1 to 43-2 | 0 (00H) | 0 – 127 |
|  | 43-3 to 86-1 | 1 (01H) | 0 – 127 |
|  | 86-2 to 99-3 | 2 (02H) | 0 – 40 |

#### 6.2.1a USER patch WRITE / EXCHANGE / INITIALIZE (app)

Saving the current (temporary) patch into a USER slot is **not** done by Bank/PC alone. The app follows the capture in `Docs/Write_SysEx.txt`:

1. **RQ1 a fresh Temporary dump** (`18 00 00 00`, size `0x3134`) immediately before WRITE — do not reuse a cached blot from an earlier dump. Live CTL/EXP/assign DT1s after that cache would otherwise be dropped and appear to “reset” after EXECUTE.
2. Send that dump as DT1s rebased onto USER wire addresses (`20 NN …` / `21` / `22`), section pages `00/03/06/07/10/20/21/30/31`.
3. Patch name is 16 ASCII bytes at Common **`+0x01`…`+0x10`** (attribute stays at `+0x00`).
4. Commit with DT1 **`0F 00 00 01 = 01`** (EXECUTE); device replies EXECUTE then **`0F 00 00 02`** payload `COMPLET…` (COMPLETE).
5. EXCHANGE swaps two USER blots then EXECUTE; INITIALIZE rebases the captured post-init temp dump (`Docs/Init00.syx` + related frames → name **Init Patch**) onto the USER slot then EXECUTE.
6. **Hardware e2e allowlist:** only **USER 09-1** and **USER 09-2** may be targeted by live device write tests (`GR408_UserWrite09HardwareTests`: write 09-1, exchange 09-1↔09-2, init 09-2).

#### 6.2.1b Library Select / Apply → Temporary (app)

Auditioning a library full patch onto the sounding Temporary Patch is **not** a USER WRITE (no EXECUTE, no slot change):

1. Prefer the stored capture blot (`Configuration.sysexData` = ordered DT1 hex frames from a prior temp or USER RQ1).
2. Rebase high address bytes to Temporary **`18 00`**, keep page/offset + payload (`UserPatchBulkWriter.rebaseDumpToTemp`); send DT1s only.
3. Sync the in-app model from the blot parse (or hydrated JSON). Select Current restores a preserved blot when one was frozen with the compare snapshot.
4. When no blot exists, fall back to catalog encode of Temporary Patch keys (`SharedParamCatalog` + `HWParameterRegistry` + `rolandDT1Address`) — reduced fidelity.
5. **Component Apply** merges the library JSON into a live temp RQ1 snapshot, then catalog-encodes only that section’s keys to Temporary (Assigns already use this path). `rng:` component slices remain hash-only and are not sent.

#### 6.2.2 **Bass Mode Decoding Table**

| Category | Patch Range | BS MSB (Hex) | PC Range (Decimal) |
| --- | --- | --- | --- |
| **LEAD** | 01-1 to 10-3 | 16 (10H) | 0 – 29 |
| **RHYTHM** | 01-1 to 10-3 | 16 (10H) | 30 – 59 |
| **OTHER** | 01-1 to 10-3 | 16 (10H) | 60 – 89 |
| **USER** | 01-1 to 43-2 | 0 (00H) | 0 – 127 |
|  | 43-3 to 86-1 | 1 (01H) | 0 – 127 |
|  | 86-2 to 99-3 | 2 (02H) | 0 – 40 |

#### 6.2.3 Crucial Distinctions

* **Preset Scaling:** In **Guitar mode**, the Lead, Rhythm, and Other categories are spread across three different MSB banks (10H, 11H, and 12H) and contain many more patches.
* **Bass Mode Compression:** In **Bass mode**, all three preset categories (Lead, Rhythm, Other) reside within the same MSB bank (**10H**) with much smaller PC ranges (30 patches each).
* **User Consistency:** The **User** bank remains consistent across both modes, using MSB 00H through 02H to cover patches 01-1 through 99-3.

### 6.3 Changing Patches

To change the active patch on the GR-55, you must send **two separate SysEx messages** in sequence:

1. **First**: Send the Bank MSB (Bank Select) to address **01 00 00 00**
2. **Second**: Send the Program Change (PC) to address **01 00 00 01**

Both messages use the DT1 (Data Set) command format.

#### 6.3.1 Bank MSB Message Format

```
F0 41 10 00 00 53 12 01 00 00 00 [MSB] [Checksum] F7
```

Where:
- `F0`: SysEx start
- `41`: Roland manufacturer ID
- `10`: Device ID (default 16)
- `00 00 53`: Model ID (GR-55)
- `12`: Command ID (DT1 - Data Set)
- `01 00 00 00`: Address (Setup Bank MSB)
- `[MSB]`: Bank MSB value (00H, 10H, 11H, 12H, etc.)
- `[Checksum]`: Calculated as `(128 - (sum of address + data bytes) % 128) % 128`
- `F7`: SysEx end

#### 6.3.2 Program Change Message Format

```
F0 41 10 00 00 53 12 01 00 00 01 [PC] [Checksum] F7
```

Where:
- `01 00 00 01`: Address (Setup Program Change)
- `[PC]`: Program Change value (0-127)
- All other bytes same as Bank MSB message

#### 6.3.3 Timing Requirements

- Send Bank MSB message first
- Wait at least **10ms** before sending Program Change message
- After sending both messages, wait **200ms** before requesting Setup data to verify the change

#### 6.3.4 Example: Changing to OTHER 01-1

To change to OTHER 01-1 (MSB 0x11, PC 112):

1. Send Bank MSB: `F0 41 10 00 00 53 12 01 00 00 00 11 6E F7`
2. Wait 10ms
3. Send Program Change: `F0 41 10 00 00 53 12 01 00 00 01 70 0E F7`
4. Wait 200ms
5. Request Setup data to verify: `F0 41 10 00 00 53 11 01 00 00 00 00 00 00 02 7D F7`

### 6.4 Summary Checklist

1. Query address **01 00 00 00** to get the **Bank MSB**.
2. Query address **01 00 00 01** to get the **Program Change (PC)**.
3. Cross-reference these two numbers with the **Patch Address Map** (image_5c73c2.png) to determine the display ID (like "User 01-1").


### 6.5 How to retrieve the Patch Bank and Patch PC from a Swift 6.2 app
To retrieve the Patch Bank and Patch PC from a Swift 6.2 app, you should send a Data Request (RQ1) SysEx message. Here is the exact RQ1 message and Swift-oriented handling for Setup Bank/PC.

#### 6.5.1 The SysEx Message
The GR-55 "Setup" area starts at address `01 00 00 00`. You can request the first 2 bytes (Bank MSB and PC) using this message:

Standard RQ1 (recommended for just the ID):

```text
F0 41 10 00 00 53 11 01 00 00 00 00 00 00 02 7D F7
```

Breakdown:

| Byte(s) | Meaning |
| :--- | :--- |
| `F0` | Exclusive status |
| `41` | Roland manufacturer ID |
| `10` | Device ID (default 16 = `0x10`) |
| `00 00 53` | Model ID (GR-55) |
| `11` | Command ID (RQ1 — Data Request) |
| `01 00 00 00` | Address (Setup area) |
| `00 00 00 02` | Length to read (2 bytes) |
| `7D` | Checksum (from Address + Length) |
| `F7` | End of Exclusive |

#### 6.5.2 Implementation in Swift
Here is a Swift 6.2 compatible snippet to calculate the checksum and build the message:

```swift
func createPatchRequest(deviceID: UInt8 = 0x10) -> [UInt8] {
    let manufacturerID: UInt8 = 0x41
    let modelID: [UInt8] = [0x00, 0x00, 0x53]
    let commandRQ1: UInt8 = 0x11

    let address: [UInt8] = [0x01, 0x00, 0x00, 0x00]
    let length: [UInt8] = [0x00, 0x00, 0x00, 0x02]

    // Checksum is calculated on (Address + Length)
    let checksumData = address + length
    let sum = checksumData.reduce(0) { (Int($0) + Int($1)) % 128 }
    let checksum = UInt8((128 - sum) % 128)

    let message: [UInt8] = [0xF0, manufacturerID, deviceID] + modelID + [commandRQ1] + checksumData + [checksum, 0xF7]
    return message
}
```

#### 6.5.3 Handling the Response
The GR-55 will respond with a Data Set (DT1) message:

```text
F0 41 10 00 00 53 12 01 00 00 00 [MSB] [PC] [Checksum] F7
```

- Byte at index 11: Patch Bank MSB (`0` = User, `16` = Lead, `17` = Rhythm, `18` = Other)
- Byte at index 12: Program Change (`0`–`127`)

#### 6.5.4 Example
Based on the message:

```text
F0 41 10 00 00 53 12 01 00 00 00 00 2B 54 F7
```

**MIDI coordinates**

| Field | Value | Notes |
| :--- | :--- | :--- |
| Bank MSB | `00` (index 11) | USER style |
| PC | `2B` (index 12) | Decimal **43** |

**UI label (`15-2`)** — patches are grouped in banks of 3; PC is 0-based:

- UI Bank = `floor(43 / 3) + 1` = `14 + 1` = **15**
- UI Number = `(43 % 3) + 1` = `1 + 1` = **2**

So Bank MSB `0`, PC `43` → **USER 15-2**.

| PC | Label |
| :--- | :--- |
| 42 | USER 15-1 |
| 43 | USER 15-2 |
| 44 | USER 15-3 |


---

## Appendix: Live Data JSON index

Parameter definitions the app loads from `GR55Configurator/Data/` (obsolete copies under `Docs/ObsoleteDocs/Data/`).

| Area | Files |
|---|---|
| Common / CTL / EXP / GK / Assigns / Master | `Patch_General`, `Patch_CTL`, `Patch_EXP`, `Patch_GK`, `Patch_Assigns`, `Patch_Master` |
| MFX | `mfx/MFXCore`, `mfx/MFXIndex`, `mfx/MFX*.json` |
| CHO / DLY / REV / EQ | `ChorusParameters`, `DelayParameters`, `ReverbParameters`, `EQParameters` |
| AMP / MOD / NS | `AmpParameters`, `mod/MODCore`, `mod/MODIndex`, `mod/MOD*.json`, `mod/MODNoiseSuppressor` |
| Modeling | `Patch_Modeling_Common`, `Patch_Modeling_Bass` |
| PCM | `Patch_PCM_Common`, `Patch_PCM_Offset`, `PCMToneNames` |
| System | `System_Common`, `System_CTL`, `System_EXP_*`, `System_GK_*`, `System_Functions` |
| Assign pickers | `AssignTargetNames`, `AssignSourceData` |

OM / settings: [Effects_Index.md](./Effects_Index.md), [Settings_Index.md](./Settings_Index.md), [Tone_Types.md](./Tone_Types.md), [ASSIGNS.md](./ASSIGNS.md).
