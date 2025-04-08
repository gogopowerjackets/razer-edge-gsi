# GSI ROM Installation Guide for Razer Edge

A guide to installing Custom GSI ROMs on the Razer Edge.

> **Note**: This guide assumes you're using Windows 11/10 and involves flashing custom software which can render your device inoperable. Backup your data first because this process involves erasing your device completely. My goal is that anyone could use this guide, but I would recommend for users to have basic command line familiarity. Proceed at your own risk!

## Table of Contents
- [Credits](#credits)
- [Initial Steps, Files, and Links](#initial-steps-files-and-links)
  - [Hardware Prerequisites](#hardware-prerequisites)
  - [Required Software](#required-software)
- [Installation Steps](#installation-steps)
  - [1. Prepare Your Razer Edge](#1-prepare-your-razer-edge)
  - [2. Install Required Tools](#2-install-required-tools)
  - [3. Unlock Bootloader](#3-unlock-bootloader)
  - [4. Extract vbmeta](#4-extract-vbmeta)
  - [5. Prepare Files](#5-prepare-files)
  - [6. Boot to Recovery and Enter Fastboot](#6-boot-to-recovery-and-enter-fastboot)
  - [7. Install GSI ROM](#7-install-gsi-rom)
  - [8. First Boot](#8-first-boot)
- [Troubleshooting](#troubleshooting)
  - [Recovery - Returning to Stock Firmware](#recovery---returning-to-stock-firmware)
- [Additional Resources](#additional-resources)

## Credits

I created this GitHub guide to make the information shared in Discord and Reddit communities easier to discover, so a very special thanks is owed to those users who originally shared their work documenting the installation process, obtaining images and files, and helping others troubleshoot:

- JoeMossJr
- npjohnson
- dpdlol
- settlefire
- defective1up
- GumbyXGames

## Initial Steps, Files, and Links

### Hardware Prerequisites

Before beginning, ensure you have:
- Unlockable Razer Edge device (WiFi or 5G model that is paid off, otherwise the bootloader cannot be unlocked)
- USB-C data cable (not just a charging cable)
  - **Note**: Razer recommends that you not use the in-box USB Type-C to Type-C charging cable for flashing. They claim that their cable is designed for power delivery, not data transfer, so to play it safe I used a USB 3 Type-A to Type-C spec-compliant cable.
- Windows 10/11 PC

### Required Software

1. **ADB and Fastboot Tools**:
   - [Latest ADB and Fastboot Installer](https://github.com/fawazahmed0/Latest-adb-fastboot-installer-for-windows) - Recommended easy installer with the latest binaries

2. **Extraction Tools**:
   - [7-Zip](https://ninite.com/7zip/) - I prefer to use Ninite to install programs when available. You'll need this for extracting firmware and ROM packages

3. **TWRP Recovery**:
   - [Download twrp-3.7.1_12-0-lahaina.img](https://dl.twrp.me/lahaina/twrp-3.7.1_12-0-lahaina.img.html) - Used for temporarily booting to extract vbmeta.

4. **GSI ROM**:
   - [LineageOS-22.1-GAPPS](https://sourceforge.net/projects/misterztr-gsi/files/LineageOS/Android%2015/LineageOS-22.1-20250319-GAPPS-EXT4-GSI.7z/download) - Android 15 with Google Apps
   - [Check Latest LineageOS GSI Releases](https://github.com/MisterZtr/LineageOS_gsi/releases) - Always get the ARM64-AB-GAPPS-EXT4 version for Razer Edge

5. **Stock Firmware** (for recovery/reversion):
   - [Razer Edge Stock Firmware Dump](https://dumps.tadiphone.dev/dumps/razer/razer-edge-wifi/tree/qssi-user-12-SKQ1.211103.001-172-release-keys/) - Official dump for reference
   - [Verizon Model Firmware (January 2025)](http://54.188.103.248:4200/resource/image/2025-01-07/cef8f6fd-1ad0-41e7-b599-801272a3fe20.zip) - For 5G models
   - [Wi-Fi Model Firmware (January 2025)](https://android.googleapis.com/packages/ota-api/package/07926a5297a8b020424230d5486ef452f666647f.zip) - For WiFi models

6. **Magisk** (optional, but required for rooting):
   - [Latest Magisk App](https://github.com/topjohnwu/Magisk/releases) - For rooting after GSI installation

## Installation Steps

### 1. Prepare Your Razer Edge

1. Make sure your battery is at least 50% charged
2. Back up any important data (the process will wipe your device)
3. Go to Settings > About Phone
4. Tap "Build Number" 7 times to enable Developer Options (you'll see a message confirming "You are now a developer!")
5. Go to back to Settings > System > Developer Options and toggle on:
   - USB Debugging
   - OEM Unlocking
6. **IMPORTANT**: Sign out of all Google accounts (Settings > Accounts)
   - This prevents Factory Reset Protection from locking you out of your device
   - Missing this step can make your device unusable after reset
7. **IMPORTANT**: Remove screen lock pattern/PIN/password (Settings > Security > Screen Lock > None)
   - This is required for proper bootloader unlocking

### 2. Install Required Tools

1. Install ADB and Fastboot:
   - Run the Latest ADB Installer from the downloaded files
   - Connect your Razer Edge to your PC 
   - Switch to "File Transfer" (MTP) mode in USB notification
   - Run Latest-ADB-Installer.bat and follow prompts to install it
   - The installer should add ADB to your system PATH
     
2. Verify installation:
   - Open a new Command Prompt window
   - Type `adb --version` and press Enter
   - You should, now or during adb installation, have gotten a request on your Razer Edge to allow connections over USB debugging
   - If you see version information, ADB is installed correctly and added to your PATH
   - If not, you'll need to navigate to your installation directory (e.g., C:\Program Files\platform-tools) to run commands

3. **Open Command Prompt**:
   - Press Win+R to open the Run dialog
   - Type `cmd` and press Enter
   - This will open a Command Prompt window where you'll run all commands in this guide

4. **Create a working directory** where all files will be stored and organized:
   ```cmd
   mkdir C:\razer-gsi
   ```

5. **Navigate to this directory**:
   ```cmd
   cd C:\razer-gsi
   ```
   To make it easier to folow the guide, all commands in this guide should be run from this directory unless otherwise specified. You can keep your command prompt open for the duration of the guide.

6. Copy TWRP to C:\razer-gsi
7. Copy the LineageOS .7z file directly to C:\razer-gsi
8. Right-click the file > 7-Zip > Extract Here

### 3. Unlock Bootloader

1. Connect device to PC
2. This is your last chance before you start copying and pasting commands to ensure you backed up your device, signed out of accounts, and removed screen lock
3. Open Command Prompt and verify connection:
   ```cmd
   adb devices
   ```
   You should see your device listed with a string of numbers/letters (device ID)
4. Reboot to bootloader:
   ```cmd
   adb reboot bootloader
   ```
   Your device will restart and display a bootloader screen with minimal UI
5. Verify bootloader connection:
   ```cmd
   fastboot devices
   ```
   You should see your device ID listed, confirming fastboot can communicate with it
6. Unlock bootloader:
   ```cmd
   fastboot flashing unlock
   ```
7. On your device screen, use volume keys to navigate to "Unlock the bootloader" and press power button
   - **WARNING:** This will factory reset your device and wipe all data
8. Wait for unlocking and reboot to complete (device will restart automatically)
9. Go through device setup again, **DO NOT** enter a Google Account or screen lock
10. Enable Developer Options and USB Debugging as you did in Step 1
11. Verify unlock in your CMD window:
    ```cmd
    adb shell getprop ro.boot.flash.locked
    ```
    (Should return "0" indicating unlocked status)

### 4. Extract vbmeta

The vbmeta file is required to disable Android Verified Boot (AVB) during GSI installation. There are several ways to obtain it:

1. Connect device to PC
2. Reboot to bootloader:
   ```cmd
   adb reboot bootloader
   ```
   Your device will restart and display a bootloader screen with minimal UI
3. **Check active slot first:**
   ```cmd
   fastboot getvar current-slot
   ```
   This will show whether you're using slot A or slot B

4. **Extract from device using TWRP:**
   - Boot (don't flash) the TWRP image (download link in File Collection section):
     ```cmd
     fastboot boot twrp-3.7.1_12-0-lahaina.img
     ```
     **Note:** The TWRP filename may change as new versions are released. Always use the filename that matches your downloaded version from the TWRP website.
   - Once TWRP loads, extract the vbmeta file for your active slot:
     ```cmd
     adb pull /dev/block/platform/soc/1d84000.ufshc/by-name/vbmeta_a C:\razer-gsi\vbmeta_a.img
     ```
     (Replace "a" with "b" if your current slot is B)

5. **Verify file was extracted:**
   ```cmd
   dir C:\razer-gsi\vbmeta_*.img
   ```
   Make sure the file exists and has a non-zero size

### 5. Prepare Files

1. Move the previously extracted vbmeta file to this working directory:
   ```cmd
   move \path\to\vbmeta_*.img C:\razer-gsi\
   ```
   Replace \path\to\ with the location where you extracted the vbmeta file

2. Verify all files are in the directory:
   ```cmd
   dir
   ```
         
   ```
   Directory of C:\razer-gsi

   03/29/2025  07:29 AM    <DIR>          .
   03/28/2025  04:31 PM     1,092,341,009 LineageOS-22.1-20250319-GAPPS-EXT4-GSI.7z
   12/31/2008  05:00 PM     3,287,248,896 LineageOS-22.1-20250319-GAPPS-EXT4-GSI.img
   03/28/2025  04:17 PM       100,663,296 twrp-3.7.1_12-0-lahaina.img
   03/28/2025  04:25 PM            65,536 vbmeta_a.img
                  4 File(s)  4,480,318,737 bytes
   ```
   
   
### 6. Boot to Recovery and Enter Fastboot

1. Boot to *recovery*:
   ```cmd
   adb reboot recovery
   ```
   
   Your device will restart into a minimal landscape recovery interface that looks different from the bootloader interface earlier
2. From recovery menu, use volume buttons to navigate, select "Enter fastboot", press the power button to select
3. Red fastbootd text will appear in the top center of the screen - this is an enhanced version of fastboot that lets us modify system partitions, so be careful here.
4. Verify connection:
   ```cmd
   fastboot devices
   ```
   
   You should see your device ID listed, confirming fastboot connection
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   C:\razer-gsi>fastboot devices
   123abca12b         fastboot
   ```
   </details>

### 7. Install GSI ROM
 
1. Wipe data:
   
   ```cmd
   fastboot -w
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Erasing 'userdata'                                 OKAY [  0.049s]
   Erase successful, but not automatically formatting.
   File system type raw not supported.
   wipe task partition not found: cache
   Erasing 'metadata'                                 OKAY [  0.004s]
   Erase successful, but not automatically formatting.
   File system type raw not supported.
   Finished. Total time: 0.101s
   ```
   </details>
   
   This erases user data partition for a clean installation
   
2. Delete logical partitions to make room for GSI:
   
   ```cmd
   fastboot delete-logical-partition product_a
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Deleting 'product_a'                               OKAY [  0.007s]
   Finished. Total time: 0.015s
   ```
   </details>

   ```cmd
   fastboot delete-logical-partition product_b
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Deleting 'product_b'                               OKAY [  0.007s]
   Finished. Total time: 0.012s
   ```
   </details>
 
   ```cmd 
   fastboot delete-logical-partition system_ext_b
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Deleting 'system_ext_b'                            OKAY [  0.007s]
   Finished. Total time: 0.013s
   ```
   </details>

   ```cmd 
   fastboot delete-logical-partition system_ext_a
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Deleting 'system_ext_a'                            OKAY [  0.007s]
   Finished. Total time: 0.013s
   ```
   </details>
 
   These commands free up space needed for the GSI ROM
   
3. Flash vbmeta (use your exact filename):

   ```cmd
   fastboot --disable-verity --disable-verification flash vbmeta vbmeta_a.img
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Rewriting vbmeta struct at offset: 0
   Sending 'vbmeta_a' (64 KB)                         OKAY [  0.003s]
   Writing 'vbmeta_a'                                 OKAY [  0.005s]
   Finished. Total time: 0.043s
   ```
   </details>
   This disables Android's boot verification to allow custom ROMs
   
4. Erase system:
   ```cmd
   fastboot erase system
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Erasing 'system_a'                                 OKAY [  0.042s]
   Finished. Total time: 0.056s
   ```
   </details>
   This removes the current system partition content
   
5. Flash GSI (this will take about two minutes):
   ```cmd
   fastboot flash system LineageOS-22.1-20250319-GAPPS-EXT4-GSI.img
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   C:\razer-gsi> fastboot flash system LineageOS-22.1-20250319-GAPPS-EXT4-GSI.img
   Resizing 'system_a'                                OKAY [  0.005s]
   Sending sparse 'system_a' 1/13 (262104 KB)         OKAY [  9.103s]
   Writing 'system_a'                                 OKAY [  1.539s]
   Sending sparse 'system_a' 2/13 (262048 KB)         OKAY [  9.180s]
   Writing 'system_a'                                 OKAY [  0.531s]
   Sending sparse 'system_a' 3/13 (262000 KB)         OKAY [  9.124s]
   Writing 'system_a'                                 OKAY [  0.542s]
   Sending sparse 'system_a' 4/13 (261960 KB)         OKAY [  9.039s]
   Writing 'system_a'                                 OKAY [  0.547s]
   Sending sparse 'system_a' 5/13 (261972 KB)         OKAY [  9.025s]
   Writing 'system_a'                                 OKAY [  0.512s]
   Sending sparse 'system_a' 6/13 (261168 KB)         OKAY [  9.234s]
   Writing 'system_a'                                 OKAY [  0.566s]
   Sending sparse 'system_a' 7/13 (262052 KB)         OKAY [  9.213s]
   Writing 'system_a'                                 OKAY [  0.553s]
   Sending sparse 'system_a' 8/13 (260147 KB)         OKAY [  9.449s]
   Writing 'system_a'                                 OKAY [  0.563s]
   Sending sparse 'system_a' 9/13 (254910 KB)         OKAY [  9.093s]
   Writing 'system_a'                                 OKAY [  0.574s]
   Sending sparse 'system_a' 10/13 (262108 KB)        OKAY [  9.088s]
   Writing 'system_a'                                 OKAY [  0.571s]
   Sending sparse 'system_a' 11/13 (262000 KB)        OKAY [  9.221s]
   Writing 'system_a'                                 OKAY [  0.544s]
   Sending sparse 'system_a' 12/13 (241146 KB)        OKAY [  8.525s]
   Writing 'system_a'                                 OKAY [  0.587s]
   Sending sparse 'system_a' 13/13 (50068 KB)         OKAY [  1.741s]
   Writing 'system_a'                                 OKAY [  0.307s]
   Finished. Total time: 126.503s
   ```
   </details>
   This writes the custom ROM to your system partition
   
6. Reboot:
   ```cmd
   fastboot reboot
   ```
   
   <details>
   <summary>Command output (click to expand)</summary>
   
   ```
   Rebooting                                          OKAY [  0.001s]
   Finished. Total time: 0.007s
   ```
   </details>
   
   Razer Logo should appear on the Edge screen and will restart with the new ROM

### 8. First Boot

- First boot may take a few minutes (be patient)
- You'll see the Android setup process with the LineageOS interface
- Complete setup process
- Done!

## Troubleshooting

If your device doesn't boot:

1. Try both bootloader and fastbootd wiping:

   <details>
   <summary>How to access bootloader mode with physical buttons</summary>
   
   If your device isn't booting normally:
   1. Power off the device completely by holding the power button for 10+ seconds
   2. Press and hold Volume Down + Power buttons simultaneously
   3. Continue holding both buttons until you see the bootloader screen (black background with minimal text)
   4. Once in bootloader mode, connect the USB cable to your PC
   5. Verify connection with `fastboot devices` in your command prompt
   </details>
   
   ```cmd
   # In bootloader:
   fastboot -w
   fastboot delete-logical-partition product_a product_b system_ext_a system_ext_b
   ```
   
   <details>
   <summary>How to access fastbootd mode from bootloader</summary>
   
   To get from bootloader to fastbootd:
   1. Make sure you're in bootloader mode first (using steps above)
   2. In your command prompt, enter:
      ```cmd
      fastboot reboot fastboot
      ```
   3. The device will reboot to a screen with red "fastbootd" text at the top
   4. Verify connection with `fastboot devices` before proceeding
   </details>
   
   ```cmd
   # Then in fastbootd:
   fastboot -w
   fastboot delete-logical-partition product_a product_b system_ext_a system_ext_b
   ```
   
   <details>
   <summary>Why this works</summary>
   
   Some users report this two-step wipe solves boot issues by ensuring all partitions are properly cleaned. The first wipe in bootloader handles standard partitions, while the second in fastbootd ensures dynamic partitions are properly handled.
   </details>

2. Wait up to 15 minutes for first boot (first boot is much slower than normal)
3. Try a different GSI ROM if still not working

### Recovery - Returning to Stock Firmware

If you need to return to stock firmware or if your device is stuck in a boot loop that can't be fixed with the steps above, you have two options:

#### Option 1: Using ADB Sideload

1. Download the [Wi-Fi Model Firmware (January 2025)](https://android.googleapis.com/packages/ota-api/package/07926a5297a8b020424230d5486ef452f666647f.zip) 

2. Boot into recovery mode:
   - Power off the device completely
   - Press and hold Volume Up + Power buttons simultaneously
   - Release when you see the recovery menu

3. From the recovery menu:
   - Use volume buttons to navigate to "Apply update from ADB"
   - Press power button to select

4. Verify device connection in recovery mode:
   ```cmd
   adb devices
   ```
   You should see your device listed with "recovery" next to it

5. Sideload the OTA package (replace with your exact filename):
   ```cmd
   adb sideload Razer-Edge-OTA-package.zip
   ```

6. After the sideload completes (100%), select "Wipe data/factory reset" from the recovery menu
   - This step is necessary when switching between ROMs
   - Confirm the factory reset when prompted

7. Select "Reboot system now" from the recovery menu

8. First boot may take several minutes. The device will automatically go through the setup process.

#### Option 2: Manual Partition Flashing (Advanced)

1. Download the appropriate stock firmware for your model (same links as above)

2. Extract the firmware zip file to your working directory:
   ```cmd
   mkdir C:\razer-stock
   cd C:\razer-stock
   ```
   Extract the downloaded zip file here using 7-Zip

3. Boot into bootloader mode (also called Download Mode):
   - Power off the device completely
   - Press and hold Volume Down + Power buttons simultaneously
   - Release when you see the bootloader screen

4. Verify device connection:
   ```cmd
   fastboot devices
   ```

5. Flash the stock firmware:
   ```cmd
   fastboot flash boot boot.img
   fastboot flash system system.img
   fastboot flash vendor vendor.img
   fastboot flash vbmeta vbmeta.img --disable-verity --disable-verification
   ```
   
   <details>
   <summary>Note about filenames</summary>
   
   The actual filenames may vary slightly based on the firmware package you downloaded. Use the exact filenames from your extracted files. Some packages might include a script to automate the flashing process (like flash_all.bat for Windows).
   </details>

6. Wipe data (this is required when switching between ROMs):
   ```cmd
   fastboot -w
   ```

7. Reboot the device:
   ```cmd
   fastboot reboot
   ```

8. First boot to stock may take several minutes. If the device gets stuck on the boot logo for more than 15 minutes, try repeating the process with a factory reset:
   ```cmd
   fastboot -w
   fastboot erase userdata
   fastboot reboot
   ```

#### Optional: Relocking the Bootloader (For Maximum Security)

After returning to stock firmware, you may want to relock the bootloader for security. Note that this will wipe your device again.

1. Boot into bootloader mode:
   - Power off the device completely
   - Press and hold Volume Down + Power buttons simultaneously
   - Release when you see the bootloader screen

2. Lock the bootloader:
   ```cmd
   fastboot flashing lock
   ```

3. Your device will show a confirmation screen. Use the volume buttons to navigate and the power button to confirm.

4. The device will reboot and perform a factory reset.

5. Complete the initial setup process.

**Warning**: Once the bootloader is relocked, you will need to unlock it again (which will wipe all data) if you want to install custom ROMs in the future.

## Additional Resources

**Community Support**:
- [Razer Edge Discord](https://discord.gg/tSpm2smb)
  - Note: If this Discord link expires, please send a message on Reddit to get an updated invite
- [Razer Edge Subreddit](https://old.reddit.com/r/razeredge_2023/)

