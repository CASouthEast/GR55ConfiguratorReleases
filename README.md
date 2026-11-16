# GR55Configurator

A macOS and iPadOS application for editing patches for the Roland GR-55 guitar synthesizer, with first-class support for the Behringer FCB1010 MIDI foot controller. The app provides a visual hardware emulation, a card-based patch editor, a hierarchical parameter tree, a local patch library, and SysEx debugging tools.

## Features

### Core Capabilities

*   **Hardware Emulation (GR55 View):** Full visual emulation of the GR-55 hardware interface
    *   Interactive pedal clusters (tap, double-tap, long press gestures) with live patch-name labels from the Level‑1 patch map
    *   Expression pedal with drag interaction
    *   Digital display with LCD aesthetic, tone/effect/assign chips, and sticky hover previews (right-click / long-press can pin a View or Edit card)
    *   **Effect structure on the LCD** — STRUCTURE is shown by MFX placement: Structure 1 (parallel) puts AMP/NS/MOD on one row and MFX on the next; Structure 2 (series) puts AMP/NS/MOD/MFX on one row
    *   **Select Patch** from the LCD name — search by name, sound style, bank (`15-2`), or full label; All vs Favorites
    *   **Control assign indicators** — CTL, EXP SW, GK S1/S2, and GK VOL show effective PEDAL/GK function plus A1–A8 badges; hover (Mac) / long-press (iPad) lists linked assign targets
    *   Navigation cluster (data wheel + surrounding arrows, knobs)
    *   **Play-mode navigation** — ← / → step patch id; ↑ / ↓ change bank; data wheel scrolls continuously across patch ids (wraps within the current sound style); wheel press cycles LEAD → RHYTHM → OTHER → USER; sound-style buttons jump to **01-1** of that style
    *   **Keyboard shortcuts & MIDI learn** — remappable maps for HW controls; inactive / live / learn modes; Shift shows shortcut badges; DIN-mode icon beside OUTPUT LEVEL (Settings)
    *   LED indicators, level meters, and sound style buttons
    *   **Hardware Edit mode** — navigate Tone / Effects / Master / System menus with the same wheel/button flow as the physical unit
    *   **STRUCT tab** — interactive signal-flow diagram for structure and per-source routing; shows `[STRUCTURE: 1]` or `[STRUCTURE: 2]` and CHO/DLY/REV sends when send levels are above zero
    *   **Looper LCD** — 4-track synchronized phrase looper on the HW display (USB audio / class-compliant I/O)
        *   Open/close: tap the **PHRASE LOOP** label under pedal 3; **EXIT** returns to play. Tap **REC/PLAY/DUB** under CTL to cycle track 1 while the looper is open
        *   **Glass header bar** — IN/OUT peak meters, live 1–4 beat / BAR readout, patch **BPM** (− / tap-edit / +, locked while transport is active), metronome **CLICK**, audio I/O status badge (tap for routing inspector), gear settings, and route refresh
        *   **2×2 track cards** (rose / teal / blue / purple) with waveform + playhead; contextual **REC → DUB → PLAY** primary and **STOP / CLEAR** secondary per track; 1-bar quantized count-in before record/overdub
        *   **macOS monitoring** — Direct Monitor forced OFF during record/overdub and OUT/THRU tests (restored after) with dry USB through-monitor; loop playback always returns on USB. **iOS** keeps GR-55 Direct Monitor ON and never through-monitors live input (hardware monitors)
        *   Bind the duplex device under **Settings → Looper Audio I/O** (also via the looper gear); OUT TEST / THRU TEST live there. MIDI-learn CC under **Settings → HW MIDI → Looper**
    *   **WRITE screen** (Owner’s Manual p.60 style) — PAGE tabs **WRITE / EXCHANGE / INITIALIZE**; dial or patch picker for USER destination; inline rename on WRITE; **WRITE** = YES, **EXIT** = cancel back to normal Display (not Edit)
        *   WRITE: temp-area dump → rebase onto USER slot → EXECUTE / COMPLETE; name injected at Common +0x01
        *   EXCHANGE: swap two USER blots (one EXECUTE per destination), then refresh display-list and pedal labels
        *   INITIALIZE: capture-faithful “Init Patch” image (`Docs/Init00.syx` / template) written to the chosen USER slot
        *   After success, Level‑1 names (patch selector + foot-pedal labels) and LCD name stay in sync via `UserPatchWriteCoordinator`
    *   Collapsible configuration tree beside the chassis

*   **Patch Editor:** Comprehensive patch editing interface
    *   Real-time patch data from a connected GR-55
    *   Card controls for sources, effects, and assigns (on/off, levels, type pickers)
    *   **Quick actions** on cards (copy/paste, level presets)
    *   **Configurable card parameters** (Settings → Card Parameters) shared with HW hover cards
    *   Effects grid with on/off status
    *   Collapsible configuration tree with **inline edit mode** (numeric / on-off / enum editors writing over MIDI)
    *   Legacy name-based UI snapshot save/load remains on the editor; primary persistence is the Library tab

*   **Patch Library, Search & Presets:** Local library of patches and reusable components
    *   **Configuration Library** — Browse, search, and filter saved configurations with grid/list modes, richer card headers (type/ID + name) and detail bodies, save/load/delete, favorites, and hash-based duplicate detection
    *   **Configurations vs Hardware Patches** — Toggle between saved configurations and the searchable live name list (browse-only; apply patches via Apply to Device from a saved configuration)
    *   **Patch Search** — Hardware View Select Patch and Library search by name, sound style (LEAD/RHYTHM/OTHER/USER), bank identifier (e.g. `15-2`), or full label, with an LRU query cache
    *   **Patch Name Map** — Factory names from JSON plus USER names via MIDI, cached to disk (24 hours); Settings → **Load USER Patches from Device** refreshes all 297 USER names
    *   **Factory Presets** — Load factory preset names and full patch details from the device with progress reporting
    *   **Auto-persist** — Patch data can be captured into the library as the device patch changes
    *   **Component Extraction** — Extract MFX, MOD, PCM 1/2, Amp, Chorus, Delay, Reverb, EQ, NS, Modeling, Assigns (with auto-generated descriptions) when loading/importing presets
    *   **Rebuild Library from SysEx** — Offline reparse of stored SysEx into JSON + component catalog without reloading from the device
    *   **Compare** — Always show Current Device + the open library item; optionally add a **Second Library** column; synced expand/collapse; **Difference Summary** of differing leaves; with three columns, pick which pair drives highlights/summary
    *   **Select / Active (full patches)** — Audition a compare column onto the GR-55 temporary area; **Active** marks what is sounding; Current stays a frozen snapshot of the working current; **Select Current** restores it. Leaving compare does **not** restore the device (so Applied changes remain for HW WRITE or further Applies)
    *   **Apply to Device** — Push a saved component into the live temporary patch (only while Current is Active — not onto an auditioned library full patch)
    *   **Version History** — Optional snapshots per configuration with restore (toggle in Settings)
    *   **Import/Export** — JSON, ZIP, and SysEx recovery dumps
    *   **CloudKit Sharing** — Share UI stub / infrastructure present; CloudKit path currently disabled

*   **Hierarchical TreeView:** Complete patch structure visualization (Patch Editor, Hardware View, Library detail)
    *   Top-level roots **Tone / Effect / Master / System** (built from the HW menu + SharedParamCatalog)
    *   Expandable/collapsible navigation; foldable pane with persistent preference
    *   View Only / Editing toggle with type-specific inline editors (live trees only; library browse is read-only)
    *   Visual status indicators and monospaced value display
    *   Library compare mode with variance highlighting

*   **Unified Parameter Catalog:** `SharedParamCatalog` / `SharedParamAccessors` load definitions from `Data/*.json` and drive hardware edit menus, tree editors, patch cards, and hover cards from one source of truth (see `Docs/HWEditParamConflicts.md` for known JSON vs legacy conflicts)

*   **Sound Source Configuration:** COSM Guitar/Bass modeling, two PCM synth tones (TVF/TVA/pitch/LFO/string levels/portamento), and Normal Guitar Pickup routing

*   **Effects Editing:** Pre-amp / amp sim, MOD (14 types), MFX (20 types), Delay, Reverb, Chorus, EQ, Noise Suppressor

*   **Control Assignments:** 8 Assign controllers, CTL, Expression, GK volume and switches
    *   **Readable assign targets & values** — assign Target names and Target Min/Max show human-meaningful content (option names, dB offsets, pan L/C/R, named pickup alternatives) instead of raw numbers, resolved through the same `SharedParamCatalog` / `SharedParamAccessors` path the individual tone/effect editors use
    *   **Single shared resolution path** — the Patch Editor, Tree view, and Hardware view all route assign display through one `AssignTargetResolver`, so every surface shows character-for-character identical target names and readable values
    *   **Context-dependent targets** — targets whose value scope depends on current patch state resolve against live context (e.g. MOD/MFX type sub-lists, and the Modeling pickup selection which follows the selected COSM model)
    *   **Hierarchical (area-first) target picker** — browse targets by area (PCM1/PCM2, Modeling, AMP, MOD, MFX, Delay/Reverb/Chorus/EQ, Patch/Master/Pickup) or select directly, keyed by the String Target_Key

*   **MIDI Integration:** Real-time device communication, SysEx parse/encode, patch request/transmission, system parameters; on iPad, a custom Bluetooth MIDI browser for WIDI / BLE MIDI
    *   Bulk USER WRITE helpers (`UserPatchBulkWriter` / `HWWriteService`) follow capture-faithful DT1 + EXECUTE/`COMPLET` sequencing (`Docs/Write_SysEx.txt`, `Docs/SysExParsing.md` §6.2.1a)
    *   Live hardware write e2e (`GR408`) is allowlisted to **USER 09-1 / 09-2** only (saves/restores originals around the run)

*   **FCB1010 Foot Controller:** Separate device with its own tabs, MIDI ports, SwiftData store, SysEx dump codec (Behringer factory firmware), global settings, 10×10 banks/presets, Preset Grid editor, local library with version history, and `.syx` / native / UnO ControlCenter **`.lgp`** import/export. Editor-only preset **name/comment** annotations are remembered by device-parameter fingerprint and restored after retrieve/import when those parameters still match
    *   **Live →GR** — When Live mode is on, presets marked →GR forward Program Change to the GR-55 on its PATCH CH (device PC RX SWITCH must be ON)
    *   **RX PC MAP picker** — From Banks & Presets / Preset Grid, pick a Guitar RX PC MAP entry to fill PC1 (and CC1 when MAP SELECT is PRG), enable →GR, and set name/comment from the map

*   **Help:** In-app Help tab summarizing functionality by sidebar area

*   **Developer Tools (optional sidebar):** Patch SysEx Debug and System SysEx Debug — toggle visibility in Settings

*   **Assign Catalog Map Maintenance (sidebar):** A review surface for the committed assign Target → Catalog mapping (`Data/AssignTargetCatalogMap.json`, 535 entries). Browse/filter by Target_Key, name, area, and match status; search the full catalog (unscoped) for a Catalog_Key; apply a match to an entry; validate the edited map; and save back, with a confirmation guard when any entry is still draft-only

### Accessibility & User Experience

*   VoiceOver / Dynamic Type / appearance support on primary hardware and navigation surfaces
*   GR55 View hover (macOS) / press-and-hold (iPad) overlays for tone, effect, and assign cards
*   Status bar, responsive scaling, smooth animations
*   Sidebar Text / Icons density toggle

## Technology

*   **Swift 6.0** with strict concurrency checking
*   **SwiftUI**
*   **macOS 26.0+** / **iPadOS 26.0+** (app target deployment currently **26.6**)
*   **Xcode 26.0+** (file-system synchronized groups)
*   **SwiftData** for local persistence (separate stores for GR-55 and FCB1010)
*   **Combine** for reactive MIDI → UI updates
*   **CoreBluetooth** (iPad) for BLE MIDI when Apple’s system browser is unsuitable


### Documentation (`Docs/`)

*   SysEx docs, address maps.

## Current Functionality

### Main Views (sidebar)

1. **GR55 View** — Hardware emulation with play-mode navigation, STRUCT Edit tab, WRITE/EXCHANGE/INITIALIZE, 4-track Looper LCD, Select Patch, control-assign indicators, keyboard/MIDI learn, hover cards, and optional tree
2. **Patch Editor** — Card UI + foldable editable tree for the live patch
3. **Library** — Saved configurations, hardware patch list, three-way compare with Select/Active audition, version history, import/export, Rebuild from SysEx, components
4. **Settings** — MIDI ports (incl. iPad Bluetooth MIDI), HW keyboard/MIDI maps, **Looper Audio I/O** (device bind + OUT/THRU tests on macOS), Load USER Patches, debug-view toggles, version history, card parameters
5. **Help** — In-app functionality summary (kept in sync with this README’s user-facing features)
6. **Assign Catalog Map** — Review, edit, validate, and save the assign Target → Catalog mapping
7. **Patch / System SysEx Debug** (optional) — Message inspection
8. **FCB1010** — Global Settings (Live →GR), Banks & Presets / Preset Grid (RX PC MAP), FCB Library (`.syx` / `.lgp`), Device

### Parsing Capabilities

Enhanced SysEx parser with 7-bit unpacking for general, modeling, PCM, amp/MOD/NS, MFX, chorus/delay/reverb/EQ, assigns, CTL/EXP/GK, and system parameters. Assign targets and their Min/Max values resolve to readable display strings via the shared parameter catalog.

### Architecture Highlights

*   Swift 6 strict concurrency (`@MainActor` / `Sendable`)
*   JSON-driven field types and address maps
*   Shared parameter catalog for HW edit, tree, and cards
*   `AssignTargetResolver` + committed `AssignTargetCatalogMap` as the single assign target/value resolution path across all three surfaces
*   `GR55StateManager` + `GR55MIDIBridge` separating MIDI from UI
*   `HWWriteService` + `UserPatchBulkWriter` for USER-slot WRITE / EXCHANGE / INITIALIZE (temp dump rebase, EXECUTE/COMPLETE wait with generation-safe timeouts)
*   Separate SwiftData stores for GR-55 library vs FCB1010
*   Extensive device-grouped automated tests (unit + optional live hardware suites such as GR405–GR408)

## GitHub

Repository: https://github.com/CASouthEast/GR55ConfiguratorReleases
Issue reporting: github.com/CASouthEast/GR55Configura…rReleases/issues/new/choose
Public project: https://github.com/users/CASouthEast/projects/2
