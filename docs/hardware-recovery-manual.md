# Mother-32 Hardware Recovery Plan

This document keeps us focused on the immediate recovery question: before we
flash any modified firmware, can we identify the Mother-32's digital hardware
well enough to understand how recovery would work?

Audience assumption: Ryan has strong Linux, security, cloud, automation, and
CLI experience, knows basic soldering and ESD practice, but is new to MCU
firmware recovery, JTAG/SWD, and debug-port discovery.

## High-Level Goal

Build confidence that a modified firmware experiment is recoverable on this
specific early-production Mother-32.

That means:

- Identify the MCU / CPU and any external memory.
- Identify the debug or programming interface, if present.
- Determine whether firmware can be read, backed up, or restored.
- Avoid relying on another unit's firmware dump unless we know it matches.
- Do not flash modified firmware until recovery is understood.

## Current Known Unknowns

Public research did not find a credible Mother-32 MCU name, debug pinout, public
service manual, or documented firmware-mod workflow.

The official update flow tells us only this:

- Firmware updates are sent by MIDI SysEx over 5-pin DIN MIDI.
- `Mother-32_ERASE_firmware.syx` is sent twice.
- The first send enters bootloader mode.
- The second send erases the old firmware and leaves the unit ready for the main
  firmware SysEx.

That strongly suggests a resident bootloader, but does not prove it is protected
from bad firmware or reachable after every failure mode.

## Immediate Next Step

The next useful step is **identify the chip and visible board interfaces**.

Do this before buying specialized JTAG/SWD adapters, flash clips, Tag-Connect
cables, or extra programmers.

### Minimum Tools for This Step

- Precision screwdriver / bit kit.
- Good lighting.
- Phone macro lens or USB microscope.
- Multimeter.
- Known-good USB MIDI interface and MIDI cable.
- Optional but useful: basic USB logic analyzer.

At this stage, the logic analyzer is useful for passive observation, not
required for opening and identifying the board.

## Step 1: Document the Board

1. Photograph the outside of the unit and serial/revision labels.
2. Open the unit using the Mother-32 manual's module-removal procedure.
3. Photograph every board, both sides if accessible.
4. Take close-ups of:
   - Largest ICs.
   - ICs near the MIDI input.
   - Any chips with 8 pins near the MCU, which could be EEPROM or flash.
   - Crystals / oscillators.
   - Unpopulated headers.
   - Rows of test pads.
   - Silkscreen labels near pads or connectors.
5. Record exact chip markings in text. Photos are useful, but search works on
   typed markings.

Stop here if anything feels mechanically risky. We can learn a lot from photos.

## Step 2: Identify Likely Digital Sections

From photos and chip markings, classify:

- Main MCU / CPU.
- External flash, EEPROM, or RAM if present.
- MIDI input/output circuitry.
- LED/button scanning circuitry.
- Voltage regulators and digital rails.
- Candidate debug connectors or test pads.

Questions to answer before probing:

- Is the MCU ARM, AVR, PIC, MSP430, TI C2000, or something else?
- Does it normally use SWD, JTAG, ISP, UPDI, Spy-Bi-Wire, or another interface?
- Does it have internal flash or rely on external memory?
- Are there visible pads that match a standard debug layout?

## Step 3: Passive Electrical Mapping

Only after visual documentation:

1. With power off, use the multimeter to find ground on connectors and pads.
2. With power off, check continuity from candidate pads to ground or obvious
   rails.
3. Power normally and measure candidate pad voltages relative to ground.
4. Mark pads as ground, power rail, idle-high digital, idle-low digital,
   toggling, reset-like, or unknown.

Do not connect a debug probe yet. The goal is classification, not interaction.

## Step 4: Decide the Recovery Path

Once the MCU is known, choose the right path:

- ARM Cortex-M: likely SWD/JTAG with OpenOCD, pyOCD, J-Link, ST-Link, or
  Raspberry Pi Debug Probe depending on vendor.
- STM32: STM32CubeProgrammer + STLINK is likely useful; readout protection rules
  matter.
- AVR: ISP/UPDI/debugWIRE path; fuse settings matter.
- PIC/dsPIC: PICkit/SNAP path; code-protection bits matter.
- TI C2000/MSP430: vendor-specific debug tools; security settings matter.
- External SPI flash/EEPROM: possible SOIC clip or in-circuit read path, but
  only after we verify voltage and bus ownership.

Do not buy target-specific tools until this decision point unless they are
generally useful anyway.

## Tools to Revisit After Chip ID

Likely useful later:

- Logic analyzer: passive signal capture and protocol decoding.
- Raspberry Pi Debug Probe: cheap SWD/UART learning tool.
- STLINK-V3MINIE: if the MCU is STM32.
- J-Link EDU Mini: broad ARM/RISC-V debug support for non-commercial use.
- SOIC flash clip / programmer: only if external flash exists.
- Tag-Connect or pogo adapter: only if pad layout matches.
- Bench supply: useful if powering boards outside the case becomes necessary.
- Soldering supplies: only if probes/pogo pins cannot make stable contact.
- Donor Mother-32 or spare main board: recommended before custom firmware on
  this early unit.

## Go / No-Go Rule

No modified firmware should be flashed until all are true:

- We know the MCU and firmware storage layout.
- We understand the official SysEx update format well enough to repack it.
- We know whether debug access exists and whether flash readout is protected.
- We have a credible way to restore this specific unit, not just a generic file.
- We have tested the tooling on non-critical hardware or a donor board where
  practical.

## Sources

- Moog Mother-32 update instructions:
  https://moogmusic-help.freshdesk.com/en/support/solutions/articles/69000840604-moog-mother-32-updating-the-firmware
- Sweetwater Mother-32 firmware update guide:
  https://www.sweetwater.com/sweetcare/articles/moog-mother-32-firmware-update/
- Moog Mother-32 user manual:
  https://www.manualslib.com/manual/988819/Moog-Mother-32.html
