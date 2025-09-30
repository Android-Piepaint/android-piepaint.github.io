---
icon: fedora
---

# Fedora installation guide

{% hint style="info" %}
Re-partitioning your device destroys android's `userdata`, make sure all important files are backed up!!
{% endhint %}

### Requirements:

* A PC with Linux installed.
* An unlocked tablet.
* Make sure `android-tools` is installed on your PC, or download `platform-tools` from [Official Website](https://developer.android.com/tools/releases/platform-tools), then decompress and cd into it.
* Download and decompress both `efi-files.zip` and `fedora-42-nabu-rootfs.img.xz` from release.
* Download ArKT-7's modded TWRP for nabu from [here](https://github.com/ArKT-7/twrp_device_xiaomi_nabu/releases/tag/mod_linux).
* Download dualboot kernel pacher from [here](https://github.com/rodriguezst/nabu-dualboot-img/releases) (If you don't know what secureboot is, just download the NOSB version).

### Partition your tablet

* Reboot your tablet into bootloader (press the power bottom and the volume down bottom together, until you see `fastboot` on screen).
*   Boot into ArKT-7's modded TWRP:

    ```
    fastboot boot path/to/downloaded/twrp/image
    ```
* Wait until your tablet to boot into TWRP, then tap on the linux logo on the top right side of the screen.
* Tap on `Partitioning` -> Enter the linux partition size -> Tap on `yes` -> Wait for partitioning to be done.

### Transferring efi file to your tablet's esp partition:

* Make sure your tablet is still in TWRP, and your tablet is still connected to PC.
*   On your PC, run:

    ```
    adb shell 'umount /esp'
    adb shell 'mount /dev/block/sda31 /esp'
    adb push path/to/unzipped/efi-file/* /esp/
    adb shell 'umount /esp'
    ```

### Install DBKP via adb sideload:

* On your tablet, go back to the home screem of TWRP.
* Tap on `Advanced` -> Tap on `ADB Sideload` -> Swipe the bar on the screen.
*   On your PC, run:

    ```
    adb sideload path/to/installer_bootmanager.zip
    ```

### Install the rootfs:

* Reboot your tablet into bootloader again.
*   On your PC, run:

    ```
    fastboot flash linux path/to/fedora-42-nabu-rootfs.img
    ```
* Wait for the process to complete, then reboot your tablet, you should see the UEFI interface.
* You can choose between boot options with volume bottom, and confirm with power bottom.
