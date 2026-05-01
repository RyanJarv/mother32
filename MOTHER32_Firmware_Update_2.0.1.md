# Moog Mother-32 Firmware Update v2.0.1

April 2021

## About Firmware Update v2.0.1

Version 2.0.1 is a minor firmware update to Mother-32. It restores the following legacy behavior that was found in firmware v1.0, but not in v2.0:

- Applying a clock or a series of triggers to the RUN/STOP input will advance the Sequencer one step per trigger, providing a quick form of basic Clock Sync.

Note: This does not impact the normal behavior of the RUN/STOP input; a regular gate signal will start and stop the sequencer as expected.

In order to begin enjoying this feature, you will first need to update your Mother-32.

## What You Will Need

- A computer with internet access, running Windows or macOS.
- A USB-MIDI Cable or Interface with 5-Pin MIDI Output.
- A program for sending MIDI SysEx data to your Mother-32. Moog recommends Bome SendSX for Windows or SysEx Librarian for macOS.

## Download & Install the SysEx Software

- Windows: <https://www.bome.com/products/sendsx/downloads>
- macOS: <http://www.snoize.com/SysExLibrarian>

## Before You Update Your Firmware

Make sure the power and MIDI connection to your Mother-32 is secure and cannot be unplugged by accident before you start the update process.

## Update Instructions

1. Connect your Mother-32 to your computer using a USB-to-MIDI cable or via your MIDI-enabled interface.

   Note: Avoid using a USB hub, if possible, as many USB hubs do not work reliably and can cause the update to fail.

2. Download the newest Mother-32 Firmware from here: <https://www.moogmusic.com/products/mother-32>.

3. Open the downloaded `.zip` file, then open the folder `Mother-32_v2.0.1`.

4. Copy these files from the folder to your SysEx application's library: `Mother-32_ERASE_firmware.syx` and `Mother-32_Firmware_v2_0_1.syx`.

5. Send `Mother-32_ERASE_firmware.syx` from your SysEx application to your Mother-32 via your USB MIDI interface. Look for the TEMPO LED to flash green/red; this means the unit is now in Boot-Loader Mode. The old firmware still needs to be erased.

6. Send `Mother-32_ERASE_firmware.syx` a second time. You should see two slow red blinks from the TEMPO LED; then, it will flash green on/off. The old firmware is now erased and the unit is ready for new firmware.

7. Send the new firmware file, `Mother-32_Firmware_v2_0_1.syx`, to your Mother-32. Observe the MIDI LED flashing red and the TEMPO LED flashing yellow during transfer.

8. Upon completion, the TEMPO LED will flash green 2-3 times and the unit will perform the normal boot-up light show sequence. Your firmware update is now complete.

If you have any questions or trouble with your update, contact <techsupport@moogmusic.com>.
