# Open Braille Beacon

An open-source, low-cost, offline BLE system that helps blind and low-vision people locate Braille signage and tactile points of interest inside public buildings when ther's not a tacktile signage in the floor
![alt text](image.png)

**V1 status:** hardware design & firmware in progress. See [Project Checklist](context/general/project-plan-checklist.md) for current progress.

---

## The Problem we solve

Many public buildings, museums, schools, hospitals, libraries have Braille signals, but with out the correct floor, they are dificult to find.
GPS like solution exists, but their are more complex and expensive, and more costly to integrate into the building than the correct floor.

Braille Beacon provides a silent, offline, low-power system that helps the user, using haptic feedback via a clip they were to locate Braille signage and points of intrest inside the building.

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
