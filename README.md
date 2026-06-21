# Open Braille Beacon

An open-source, low-cost, offline BLE system that helps blind and low-vision people locate Braille signage and tactile points of interest inside public buildings when ther's not a 

**V1 status:** hardware design & firmware in progress. See [Project Checklist](context/general/project-plan-checklist.md) for current progress.

---

## The Problem

Many public buildings — museums, schools, hospitals, libraries — have Braille signage, tactile walking surface indicators (tactile paving), and other tactile information points, but they can be difficult for blind and low-vision visitors to find. Existing navigation solutions are either too expensive, require cloud connectivity, or are simply not designed for this use case.

Open Braille Beacon provides a silent, offline, low-power cue system that works alongside existing tactile signage — not as a replacement for a white cane, guide dog, or mobility training.

## How It Works

The system consists of two devices that communicate over Bluetooth Low Energy (BLE):

```
┌──────────────────────┐        BLE Advertisement        ┌──────────────────────┐
│   Wall Beacon Tag    │ ──────────────────────────────► │   Receiver Clip      │
│                      │   compact ID + metadata         │   (wearable, haptic) │
│ • BLE module (nRF52) │                                 │   • nRF52840 MCU     │
│ • CR2032 battery     │                                 │ • LiPo battery       │
│ • 3D-printed housing │                                 │ • Vibration motor    │
│ • Reed switch        │                                 │ • Single button UI   │
└──────────────────────┘                                 └──────────────────────┘
```

### Tag → Receiver Flow

```
OFF ──► ROOM SCAN ──► TAG SELECTION ──► GUIDE MODE ──► ARRIVED / STANDBY
```

1. **Tags** silently broadcast a compact BLE advertisement containing a tag ID, type, and zone metadata.
2. **Receiver** scans the area and presents detected tags to the user.
3. User selects **one** destination (short press cycles tags, long press confirms).
4. Receiver enters guide mode — haptic feedback changes as the user gets closer:
   - **Slow vibration** → far
   - **Faster vibration** → near
   - **Distinct pattern** → arrived
5. Long press cancels guidance at any time.

### Key UX Principle

Many tags must **not** create many alerts. The receiver guides toward **one selected tag only**; all others are ignored during guide mode.

## ⚠️ Safety & Scope

This is an **assistive location cue**, **not** certified indoor navigation. It must **never** replace:

- A white cane or guide dog
- Orientation and mobility training
- Physical tactile signage
- A user's safety judgment

BLE RSSI is approximate proximity only — it does not provide exact distance, direction, alignment, or obstacle detection.

## System Components

### Wall Beacon Tag

| Component | Part |
|---|---|
| BLE module | Raytac MDBT42Q (nRF52832) |
| Battery | 1× CR2032, 3 V |
| Switch | Magnetic reed switch |
| Enclosure | 3D-printed PETG/PLA (~40×35×12 mm) |
| Mounting | 3M VHB adhesive pad |

- Silent BLE advertisement, no audio
- Battery life target: months of intermittent use
- Reed switch for service/wake control
- Compact, discreet wall-mounted form factor

### Receiver Clip

| Component | Part |
|---|---|
| MCU | Seeed Studio XIAO nRF52840 |
| Battery | 150–300 mAh protected LiPo |
| Haptics | 10 mm coin vibration motor (MOSFET-driven) |
| Input | Single tactile button + power switch |
| Enclosure | 3D-printed wearable clip |

- Wearable clip-on form factor
- Haptic-only feedback (no audio in v1)
- USB-C charging (via XIAO board)
- Motor driven through MOSFET — **never** directly from a GPIO pin

## Repository Structure

```
├── AGENTS.md                          Contribution guidelines & design rules
├── cad/
│   ├── cadquery/                      Parametric enclosure models (CadQuery)
│   │   ├── top_cover.py               Top shell with LED window
│   │   ├── backplate.py               Base plate with screw bosses, VHB recess
│   │   └── assembly.py                Full assembly with component placeholders
│   └── exports/                       Generated STEP/STL files (auto-generated)
├── code/
│   ├── beacon_code/                   Tag firmware (Zephyr/nRF Connect)
│   └── reciever-code/                 Receiver firmware (Zephyr/nRF Connect)
├── context/
│   ├── general/                       Project description & planning
│   │   ├── project-description.md
│   │   └── project-plan-checklist.md
│   └── hardware/                      Hardware design notes & red flags
│       ├── electrical-red-flags.md
│       └── components/                BOM and component datasheets
└── protocol/
    └── protocol-v0.1.md               BLE advertisement specification
```

## Protocol

Tags use a compact BLE advertising format. Key fields:

- `project_id` / `protocol_version` — receiver filter
- `tag_id` — unique identifier
- `type` — `BRAILLE_SIGN`, `EXIT`, `DOOR`, `STAIRS`, `ELEVATOR`, `INFO_POINT`, etc.
- `room_id_hash` / `zone` — location context
- `pair_id` / `pair_role` — paired tags for doorways/stairs
- `battery_level` — remaining charge

RSSI smoothing uses exponential weighted average: `smoothed = 0.8 × prev + 0.2 × current`

See [protocol-v0.1.md](protocol/protocol-v0.1.md) for the full specification.

## CAD Models

Parametric CadQuery models for the tag enclosure. Exports go to `cad/exports/`.

```powershell
python cad/cadquery/top_cover.py
python cad/cadquery/backplate.py
python cad/cadquery/assembly.py
```

Design targets: 40×35 mm footprint, ~12 mm assembled height, 2 mm wall thickness, 0.3 mm print clearance.

See [cad/README.md](cad/cadquery/README.md) for details.

## Current Status

### Phase 0 — Foundation ✅

- Product boundary, safety language, UX, tools, and core parts frozen.

### Phase 1 — Wall Beacon Tag 🔄

- CAD models defined, fabrication and bench testing pending.

### Phase 2 — Receiver Prototype 🔄

- Components selected, hardware build pending.

### Phase 3 — Firmware 📋

- Protocol defined, firmware implementation pending.

### Phase 4 — Verification 📋

- System testing and documentation pending.

### Phase 5 — Pilot Study 📋

- Controlled accessibility study planned for v2.

## Design Rules

- **Offline-first:** no cloud services in v1.
- **Single CR2032** for the tag — never two in series.
- **MOSFET/driver** for the vibration motor — never a GPIO pin directly.
- **BLE antenna keep-out:** no copper, battery, or metal over the antenna area.
- **Parametric CAD:** all dimensions in millimeters; regenerate exports, don't hand-edit.
- **MVP focus:** practical, low-power, low-cost, open-source.

## License

See [LICENSE](LICENSE) for details.

---

*Open Braille Beacon is an assistive tool, not a safety device. Always prioritize user safety and judgment.*
