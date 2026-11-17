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
