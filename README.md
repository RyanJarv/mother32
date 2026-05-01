# Mother-32 Firmware v2.0.1 Notes

This repository contains the Moog Mother-32 v2.0.1 firmware update files and
supporting Moog PDFs. The working goal is to understand the update format and
firmware behavior well enough to add a practical sequence-inspection mode.

## Goal: View the Current Sequence Note

The Mother-32 sequencer makes it hard to correct a single note after a sequence
has been entered: if the wrong pitch is not obvious by ear, the practical fix is
often to re-enter the sequence. The feature target is a mode that shows the
stored pitch for the currently playing sequencer step as playback advances.

Desired behavior:

- The mode should be read-only: it must not alter notes, rests, accents, glide,
  sequence length, playback direction, tempo, or clock behavior.
- While a sequence plays, the UI should indicate the current step's stored note.
- The display should follow existing sequencer playback behavior, including
  forward, reverse, pendulum, random, and externally clocked advancement.
- The mode should be easy to enter and leave from the existing panel controls.
- Firmware v2.0.1's restored `RUN/STOP` trigger behavior must be preserved.

Open product-design questions:

- Which existing indicator should show pitch: keyboard LEDs, step LEDs,
  octave/location LEDs, tempo LED color/state, or a combination?
- Should the mode show absolute stored note, transposed output note, octave, or
  both pitch class and octave?
- Should the mode run during playback only, or also allow manual step-by-step
  inspection while stopped?
- What button gesture can be added without conflicting with existing keyboard,
  step, playback-mode, and setup functions?

## Files in This Repository

- `Mother-32_Firmware_v2_0_1.syx`: the v2.0.1 firmware payload distributed as
  a MIDI System Exclusive file.
- `Mother-32_ERASE_firmware.syx`: the erase / boot-loader command that Moog's
  update instructions send twice before the firmware payload.
- `MOTHER32_Firmware_Update_2.0.1.pdf`: one-page official update instructions.
- `Mother-32-Exploration-Patchbook-Firmwarev2.0.pdf`: Moog patchbook for
  firmware v2.0, including short sequencer workflow notes.
- `assets/patchbook/`: extracted patchbook images.

## Firmware Update Behavior from Moog's PDF

Moog's v2.0.1 update is a minor update over v2.0. It restores legacy firmware
v1.0 behavior where applying a clock or trigger series to the `RUN/STOP` input
advances the sequencer one step per trigger. A regular gate on `RUN/STOP` still
starts and stops the sequencer.

The documented update flow is:

1. Send `Mother-32_ERASE_firmware.syx` once. The unit enters boot-loader mode;
   the `TEMPO` LED flashes green/red.
2. Send `Mother-32_ERASE_firmware.syx` a second time. The old firmware is
   erased; the `TEMPO` LED gives two slow red blinks, then flashes green.
3. Send `Mother-32_Firmware_v2_0_1.syx`. During transfer, the `MIDI` LED
   flashes red and the `TEMPO` LED flashes yellow.
4. On completion, the `TEMPO` LED flashes green two or three times and the unit
   performs its normal boot-up light sequence.

## Observed SysEx File Format

These observations are from the two `.syx` files in this repository. They are
not yet a complete specification.

Both files are single MIDI System Exclusive messages:

- Start byte: `0xf0`
- Manufacturer ID: `0x04` (Moog)
- Observed product / device byte: `0x14`
- End byte: `0xf7`
- All bytes inside the SysEx envelope are 7-bit clean; no data byte outside
  `0x00` through `0x7f` appears between `0xf0` and `0xf7`.

### Erase File

`Mother-32_ERASE_firmware.syx` is exactly five bytes:

```text
f0 04 14 11 f7
```

Working interpretation:

- `0x04`: Moog manufacturer ID.
- `0x14`: Mother-32 product / target ID, inferred because both update files
  share it.
- `0x11`: erase / boot-loader command, inferred from Moog's update procedure.

### Firmware File

`Mother-32_Firmware_v2_0_1.syx` is 77,219 bytes:

```text
f0 04 14 00 00 01 00 00 00 00 00 00 01 00 02 00 ... f7
```

Observed structure:

- It contains one `0xf0` byte at offset `0` and one `0xf7` byte at offset
  `77218`.
- The command byte after `f0 04 14` is `0x00`, inferred to mean firmware data.
- After the leading bytes, the payload has a strong repeating 3-byte packing
  pattern:
  - First byte of each packed group is usually `0x40` through `0x4f`.
  - Second and third bytes are `0x00` through `0x3f`.
  - This suggests a 7-bit-safe encoding for 16-bit words:

```text
word = ((byte0 & 0x0f) << 12) | (byte1 << 6) | byte2
```

Using that working decode after the apparent header produces 25,734 16-bit
words, or 51,468 decoded bytes. The end of the payload decodes to zero padding.

Current unknowns:

- Exact header field meanings.
- Whether the decoded 16-bit words are executable code, packed data, encrypted
  data, compressed data, checksummed blocks, or a mixture.
- Target CPU / memory map.
- Checksum, block, and address rules used by the boot-loader.
- Whether the payload embeds a firmware version field beyond the file name and
  update PDF.

## Relevant Mother-32 Sequencer Functionality

From the local Moog PDFs, the relevant documented behaviors are:

- `SHIFT` + `KB` enters keyboard mode.
- `OCTAVE/LOCATION` buttons select keyboard octave.
- Basic sequence workflow in the patchbook is: clear sequence, record sequence,
  enter notes, finish recording, begin playback.
- Assignable output mode `1` is Accent.
- Assignable output mode `8` is Sequencer Step Random.
- Playback mode shortcuts shown in the patchbook:
  - Hold `KB` + `STEP`, then press `STEP 2` for reverse playback.
  - Hold `KB` + `STEP`, then press `STEP 3` for pendulum playback.
  - Hold `KB` + `STEP`, then press `STEP 4` for random playback.
- Firmware v2.0.1 restores trigger-per-step advancement from the `RUN/STOP`
  input while preserving normal gate start/stop behavior.

Firmware areas likely relevant to the goal:

- Sequencer step advancement and playback-direction logic.
- Sequence event storage: note, octave, rest, accent, glide, and sequence length.
- Keyboard / step LED driving code.
- Button chord handling for `SHIFT`, `KB`, `STEP`, and numbered step buttons.
- External clock or `RUN/STOP` trigger handling.
- Any display refresh path that runs once per sequencer step.

## Reverse-Engineering Checklist

1. Confirm the SysEx command/header format by comparing with other Mother-32
   firmware versions if available.
2. Write a small decoder for the observed 3-byte-to-16-bit packing.
3. Identify the target CPU and instruction set from the Mother-32 hardware or
   decoded payload.
4. Locate interrupt vectors, reset handler, or boot metadata in the decoded
   payload.
5. Locate button scan, LED output, and sequencer state routines.
6. Map the in-memory sequence representation.
7. Find a low-risk UI hook for a read-only current-note display mode.
8. Repack a byte-identical firmware SysEx after decode/encode round-tripping.
9. Only then attempt behavioral patches, with a recovery plan for failed
   flashes.

## Safety Notes

Firmware patching can brick the instrument if the payload, packing, checksum, or
boot-loader expectations are wrong. Keep the original `.syx` files unchanged,
work on copies, and verify that any unpack/repack process is byte-identical
before making functional changes.

Before flashing any modified firmware, follow the hardware recovery plan in
[`docs/hardware-recovery-manual.md`](docs/hardware-recovery-manual.md).
