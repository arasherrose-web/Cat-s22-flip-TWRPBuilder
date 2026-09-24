# CAT S22 Flip Android 11 TWRP build (experimental)

This guide is for the CAT S22 Flip Android 11 device tree in this repository. This is an unofficial experiment, not an official Team Win release. A recovery image built locally was temporarily booted on one handset, but partition mounting and the return-to-Android behavior are not fully validated. Do not treat it as ready for routine recovery, ROM installs, or backups.

## Build on GitHub

1. Open this repository's **Actions** tab.
2. Select **Build experimental TWRP for CAT S22 Flip (Android 11)**, then **Run workflow**.
3. Leave **bcb_fallback** unchecked for the normal build. It is an experimental option described below.
4. When the run finishes, download the cat-s22-flip-twrp-android-11 artifact from that run. GitHub keeps the artifact for seven days.

The workflow uses the TWRP 11 minimal manifest and the CAT S22 Flip Android 11 device tree. It changes the device-tree product prefix from omni to twrp, appends the device-tree dtb.img to the prebuilt kernel, and removes the empty separate-DTB mkbootimg argument. This packaging change was used for a local image that reached the TWRP interface with fastboot boot on one CAT S22 Flip. The GitHub workflow itself still needs a successful run before its artifact can be considered verified.

## Test before considering a flash

Use Android platform-tools on a computer, connect the phone, and make sure its bootloader is already unlocked. Unlocking a bootloader commonly erases phone data.

From PowerShell, after replacing the image path if needed, run these commands:

$pt = "C:\Users\$env:USERNAME\AppData\Local\Android\Sdk\platform-tools"
& "$pt\adb.exe" reboot bootloader
& "$pt\fastboot.exe" devices
& "$pt\fastboot.exe" boot "$env:USERPROFILE\Downloads\recovery.img"

fastboot boot tests the image from memory and does not write the recovery partition. Check that the screen and touch work. In the test handset's TWRP log, /system_root failed to mount with “Invalid argument,” and a backup attempt failed. Do not rely on this build to back up or restore data, install ROMs, or repair a phone. If the test fails, use fastboot reboot to return to Android when possible.

Do not flash this image just because the temporary boot reaches the TWRP screen. A successful temporary boot proves only that the bootloader can start the image; it does not prove all partitions work or that Android will boot correctly afterward. This guide does not recommend a fastboot flash recovery command until the device's partition layout and recovery behavior are confirmed for that exact handset.

## Optional BCB fallback (experimental)

The workflow input bcb_fallback applies a change to TWRP's bootloader-message clearing code. It tries the normal clear first, then falls back to /dev/block/mmcblk0p27 if that fails. That path came from one CAT S22 Flip's recovery log, where /misc mapped to mmcblk0p27. Partition numbers can differ by device, firmware, or variant; using the wrong path can damage data or prevent booting. Keep the option off unless your own recovery log confirms the same mapping and you understand the risk.

The fallback has not been confirmed to resolve the system-reboot loop reliably. On the test phone, erasing misc in fastboot let Android start again, but that does not establish a universal fix. Test any fallback image with fastboot boot first and do not flash it based solely on this report.

## Known limitations

- TWRP logged failed to read default fstab and failed to mount /system_root on the test phone.
- A TWRP backup failed on that handset; encrypted storage and other mounts have not been validated.
- The DTB packaging was tested by temporarily booting a locally built image on one handset. It has not been confirmed across CAT S22 Flip variants.
- The optional BCB fallback is experimental and based on one handset's partition map.
- GitHub Actions produces a build artifact, not a supported or signed release.

Please report the exact handset variant, Android build, workflow run link, and relevant recovery log lines when reporting a problem. Never post passwords, account tokens, or private data from logs.
