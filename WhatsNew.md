## What's New (since 0.8.84)

### Looper
- **Sample-exact sync & 4-bar timing** — Replaced player-node scheduling with a sample-exact render clock and host-timestamp input alignment:
  - `4, 3, 2, 1` countdown starts exactly one bar ahead of record/overdub and locks cleanly to the beat grid.
  - Recording always completes full bars (4/4 default, with groundwork for future rhythms) and keeps steady tempo across loop repeats.
  - All 4 tracks stay phase-locked in Record, Dub, and Play; Play waits for bar start (and Loop 1's starting point for tracks 2–4), and stopping Loop 1 stops all 4 tracks.
- **Unity capture & Output Level knob** — Fixed multi-channel USB input folding (no +6 dB channel summing when channels 1–2 carry signal) and connected the **OUTPUT LEVEL** knob exclusively to the looper's overall output level (no MIDI/SysEx sent).
- **Click output routing** — Configure **Click Device** (macOS) and **Click Channel** (macOS/iOS when multiple output channels are available) in Looper Settings to send the click to a separate device or channel in sync with the looper.
- **MIDI Learn & view gating** — Added MIDI Learn for Looper 1–4 (`REC/PLAY/DUB`, `STOP`, `CLEAR`) and **Click On/Off** in both the looper UI and MIDI Controller Settings. **PHRASE LOOP** (under Foot Switch 3) shares the Loop 1 `REC/PLAY/DUB` MIDI binding when on the normal Play display to open the looper view without starting recording; all other looper MIDI commands are disabled outside the looper view.
- **Light-mode contrast & bar/beat markers** — High-contrast waveform wells, perceptual waveform scaling, clearer `CLEAR` button contrast in Light Mode, and bar/beat markers inside each loop's waveform view.


## What's New (since 0.8.83)

### Library
- **Snapshots in Library** — Save/load the working-patch clipboard from the Patch Editor into Library category **Snapshot**. Save does **not** WRITE a USER slot; Load goes to the temporary area. Older editor snapshots migrate automatically once.

### Patch editing consistency
- **Assign Min/Max labels** match across Patch Editor, Tree, Hardware hover, closed Assign cards, and HW Edit LCD.
- **Tree edits** for catalog parameters write to the device immediately (same timing as cards and HW Edit).
- **Stored vs sounding switches** — Tree and HW Edit show stored switch bits; cards, STRUCTURE, and LEDs may show what is actually sounding when CTL/EXP TONE SW overlays apply. This is intentional.

### Hardware Edit
- Tap Edit / WRITE tabs and menu rows to navigate.
- Drag sideways on a selected value to scrub like the data wheel.

### Looper (still alpha)
- Better recovery when USB audio reconfigures, the input tap goes silent, or recording stalls mid-session.
- Prefer **GR-55 USB** under Settings → Looper Audio I/O.

### App
- Short branded launch splash showing the build version.

### Tester note
If Tree switch values disagree with card LEDs, check CTL/EXP TONE SW overlays before filing a bug.
