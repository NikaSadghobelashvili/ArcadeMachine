# Arcade (Tabletop Hybrid Arcade Cabinet) — Project Doc

This repository contains documentation for a tabletop **hybrid arcade cabinet** project (Junior Project, 2025) developed at **Free University of Tbilisi (MACS[E])**.

The original project report is in `ArcadeDoc.pdf`.

## What this is

The system is a compact tabletop arcade cabinet designed to recreate a classic arcade experience while supporting **multiple input/control modules** within one device:

- Joysticks + buttons (multiplayer)
- Drum pads (shock sensors)
- Potentiometer “paddle/knob” controller (Arkanoid/Breakout-style)
- A (wireless) gun-style controller (magnetometer-based aiming)

It runs a mix of:

- **Custom games** (developed for Raspberry Pi)
- **Retro games** through emulators (e.g., classic console titles / MAME)

## High-level architecture

- **Compute**: Raspberry Pi 5 (main system)
- **Input handling**:
  - USB encoder(s) for joystick/button inputs
  - Arduino boards for sensor reading and HID emulation (keyboard/mouse)
  - nRF24L01 modules for wireless gun controller data transfer
- **Launcher/UI**: Python + Pygame fullscreen menu that launches games and returns to menu via a “Home” button flow

## Hardware/modules (from the report)

- **Raspberry Pi 5**
- **Arduino Uno / Leonardo** (multiple boards used)
- **nRF24L01 (NRF24L01+)** wireless transceiver modules
- **GY-87 sensor module** (accelerometer/gyroscope/magnetometer used in the gun controller design)
- **Shock sensors**: HW-072 and SW-420 (drum pad impact detection)
- **Potentiometer** (analog paddle control)
- **Display adapter**: HDMI-to-VGA (Pi HDMI output to VGA display)
- **Audio adapter**: 3.5mm-to-USB (Pi 5 lacks analog audio out)
- **Fabrication**: Fusion 360 design, CNC-cut panels, 3D-printed parts

## Software stack

- **Game development**: GameMaker Studio 2 (custom games compiled for Raspberry Pi OS)
- **Hardware interaction / glue**:
  - Arduino sketches (sensor reading, HID emulation, nRF24 comms)
  - Python scripts (menu + game launching)
- **Input mapping**: AntiMicroX used to map some USB encoder/controller inputs to virtual controller inputs (needed for custom games’ controller handling)

## Controls overview

- **Joysticks & buttons**: Through USB encoder; mapped as needed via AntiMicroX profiles.
- **Drum pads**: Shock sensors routed to an Arduino Leonardo that emulates keyboard keypresses (report example types `'1'` and `'2'`).
- **Potentiometer**: Read by Arduino; mapped to mouse movement for smooth analog paddle control.
- **Gun controller**:
  - Transmitter reads sensor orientation, computes deltas vs. initial “center”, sends small `int8` deltas over nRF24.
  - Receiver (Leonardo) converts received deltas into `Mouse.move()` calls and handles mouse click state.

## Menu / launcher behavior (report listing)

The menu is a fullscreen Pygame UI that:

- Autostarts at boot (via a `.desktop` entry described in the report)
- Lets users select from a list of games
- Launches a selected game via shell command
- Reinitializes the menu after the game exits

The report also describes a dedicated **Home/Exit** button intended to exit a running game and return to the menu cleanly.

## Code listings included in `ArcadeDoc.pdf`

The PDF includes example code listings (embedded in the report) for:

- **Gun master (UNO transmitter)**: GY-87 reading + nRF24 payload transmit
- **Gun receiver (Leonardo)**: nRF24 receive + USB mouse emulation
- **Sensors + potentiometer + Home button**: interrupts + keyboard/mouse HID
- **Python menu/launcher**: Pygame UI + `subprocess` launching + AntiMicroX profile startup

If you want these extracted into real source files (e.g. `arduino/` + `menu/` folders) so they can be built/edited directly, tell me and I’ll reconstruct them into compilable projects.

## Project goals (as stated in the report)

- Combine multiple control schemes in one cabinet (digital + analog + motion-based)
- Support both custom-made games and emulated retro titles
- Provide a simple game selection UX with a persistent menu loop
- Keep the build compact and aesthetically clean (wireless where possible)

## Notes / key implementation challenges (from the report)

- **Gun controller tracking**:
  - Multi-magnetometer approach was blocked by identical I2C addresses and interference from metal/electronics.
  - Final approach used a **single magnetometer** and relative deltas from an initial reference orientation.
- **Input recognition in custom games**:
  - Some raw USB inputs were not recognized by the custom game’s controller library.
  - **AntiMicroX** was used to map inputs to virtual controller inputs.
- **Drum sensor noise**:
  - Shock sensors produced noisy/multiple triggers.
  - Mechanical damping (springs) was used to reduce oscillation.

## References

See the “References / Used Literature” section in `ArcadeDoc.pdf`, including Arduino HID docs, Pygame, GameMaker manual, Raspberry Pi docs, and nRF24L01 datasheet sources.
