---
description: Another installation guide for newer Ubuntu port.
icon: ubuntu
---

# Ubuntu installation guide ( New method )

This document is a guide for installing Ubuntu Linux on your Xiaomi Mi Pad 5 with the latest mainline kernel.

[_**We don’t take any responsibility for any damage done to your device. By following this guide, you agree to take full responsibility of your actions.**_](#user-content-fn-1)[^1]

{% hint style="info" %}
**PLEASE READ SLOWLY AND CAREFULLY!! BE SURE TO UNDERSTAND THE ENTIRE GUIDE BEFORE STARTING!!**&#x20;

**Don’t run all commands at once** and **don’t rerun the commands if you interrupt the process**. You  **need** to be familiar with command line interfaces beforehand and you **must not** commit any typo with any commands. _You may permanently break your device!_
{% endhint %}

### Fork the repository

This repository is currently maintained by map220v on GitHub:&#x20;

{% embed url="https://github.com/map220v/ubuntu-xiaomi-nabu" %}

Open your browser and search GitHub, then login your account, click the link above.&#x20;

<figure><img src="../.gitbook/assets/Screenshot From 2025-01-21 22-27-36.png" alt=""><figcaption><p>A screenshot showing map220v's GitHub repository.</p></figcaption></figure>

Click the "Fork" button to fork this repository. Once it forked, head to the next part of this guide, we'll build a custom system image for our further installation.

## Build system image

Thanks to GitHub Actions, the build sections are all finished automatically, so we don't need to compile them manually. After you forked this repository, go to "Actions" tab, find the "rootfs" section in sidebar, then click "Run workflow" to build a kernel and system image. It will takes you about 23 minutes and 42 seconds to finish this build.

<figure><img src="../.gitbook/assets/Screenshot From 2025-01-21 22-41-48.png" alt=""><figcaption><p>The Actions sections showing in my forked repository.</p></figcaption></figure>

Once the build process is finished, click the "rootfs" in your workflow table. Scrolling down the page, you'll find the "Artifacts" section.

Here are your build files, download the `xiaomi-nabu-debs_6.7-working` and rootfs package. Either `rootfs_lomiri_6.7-working`  or `rootfs_ubuntu-desktop_6.7-working` is OK.

{% hint style="danger" %}
**WARNING**: **DO NOT** download 6.8 packages as 6.8 kernel [is known to have a broken screen framebuffer](https://xdaforums.com/t/rom-ubuntu-on-xiaomi-pad-5-nabu.4597149/post-89903309), it will show nothing on screen (even the console) after you installed them.
{% endhint %}

From now on, follow the [old installation guide](install-ubuntu-on-your-tablet-legacy-images-only.md) until the "Install new system" chapter.



### Partition the UFS



{% hint style="success" %}
This procedure will not erase your Android data files, since Android will automatically fixing the `userdata` partition.
{% endhint %}

To modify the partitions on the UFS, we'll need to download a 3rd-party recovery environment called "Orangefox Recovery" Link is here below:

{% embed url="https://drive.google.com/file/d/1gCNtoDMNCAmMR61xegvCC3mxv28gMJbi/view?usp=drive_link" %}

Once you've downloaded, open a terminal and type the following commands:

```bash
adb reboot bootloader
fastboot boot /path/to/recovery.img 
```

This will start booting recovery image. Once the screen is on, use the terminal to continue processing. We well use `adb shell` command to finish the rest of this guide. Enter this command in your terminal, it well help you to check the userdata partition's location:

```bash
ls -l /dev/block/bootdevice/by-name/ | grep userdata
lrwxrwxrwx 1 root root 16 1971-06-22 05:04 userdata -> /dev/block/sda31    #Example output will like this.
```

In this example, the `userdata` partition is located in the 31st partition of the whole disk. It has the biggest size, compared with other partitions. So this is the key for requiring new space for our Ubuntu installation.&#x20;

To resize the `userdata` partition, we'll need to use the [parted](https://renegade-project.tech/tools/parted.7z) command tools to do this. Now let's open `adb shell` again and start typing `parted` in the terminal.

```bash
parted /dev/block/sda
GNU Parted 3.3
Using /dev/block/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

Enter `print` command to list all partitions for `/dev/block/sda` :



{% hint style="info" %}
If you are using newer recovery like [TWRP](https://sourceforge.net/projects/xiaomi-pad-5/files/TWRP/Android%2014/) or PBRP Recovery, you need to download `parted` and use `adb` command to copy it to internal storage. Run these commands to finish this:

```bash
adb push parted /tmp/
adb shell chmod 755 /tmp/parted
adb shell
```
{% endhint %}

Then you will see your current partition table with **`userdata` being the last partition**\
Below is an example of output:

```bash
(parted) print
print
Model: SAMSUNG KLUDG4UHDC-B0E1 (scsi)
Disk /dev/block/sda: 126GB
Sector size (logical/physical): 4096B/4096B
Partition Table: gpt
Disk Flags:
Number  Start   End     Size    File system  Name             Flags
 1      24.6kB  32.8kB  8192B                switch
 2      32.8kB  65.5kB  32.8kB               ssd
 3      65.5kB  98.3kB  32.8kB               dbg
 4      98.3kB  131kB   32.8kB               bk01
 5      131kB   262kB   131kB                bk02
 6      262kB   524kB   262kB                bk03
 7      524kB   1049kB  524kB                bk04
 8      1049kB  1573kB  524kB                keystore
 9      1573kB  2097kB  524kB                frp
10      2097kB  4194kB  2097kB               countrycode
11      4194kB  8389kB  4194kB               misc
12      8389kB  12.6MB  4194kB               vm-data
13      12.6MB  16.8MB  4194kB               bk06
14      16.8MB  25.2MB  8389kB               logfs
15      25.2MB  33.6MB  8389kB               ffu
16      33.6MB  50.3MB  16.8MB               oops
17      50.3MB  67.1MB  16.8MB               devinfo
18      67.1MB  83.9MB  16.8MB               oem_misc1
19      83.9MB  101MB   16.8MB  ext4         metadata
20      101MB   134MB   32.9MB               bk08
21      134MB   168MB   34.2MB               splash
22      168MB   201MB   33.6MB               bk09
23      201MB   9328MB  9127MB               super
24      9328MB  9328MB  131kB                vbmeta_system_a
25      9328MB  9328MB  131kB                vbmeta_system_b
26      9328MB  9396MB  67.1MB               logdump
27      9396MB  9530MB  134MB                minidump
28      9530MB  9664MB  134MB                rawdump
29      9664MB  10.7GB  1074MB  ext4         cust
30      10.7GB  10.9GB  134MB   ext4         rescue
31      10.9GB  126GB   115GB                userdata
```

Now let’s continue partitioning:\
Here the size of `userdata` can be decided by yourself.

Delete partition 31 and again make sure it is not deleted incorrectly.

```bash
(parted) rm 31
rm 31
```



{% hint style="info" %}
126GB is the `End` value for the new `userdata` partition.\
Since the starting point for `userdata` is 10.9GB, the new size would be `126G - 10.9G = 115G`.
{% endhint %}

Check the results:

```bash
(parted) print
print
Model: SAMSUNG KLUDG4UHDC-B0E1 (scsi)
Disk /dev/block/sda: 126GB
Sector size (logical/physical): 4096B/4096B
Partition Table: gpt
Disk Flags:
Number  Start   End     Size    File system  Name             Flags
 1      24.6kB  32.8kB  8192B                switch
 2      32.8kB  65.5kB  32.8kB               ssd
 3      65.5kB  98.3kB  32.8kB               dbg
 4      98.3kB  131kB   32.8kB               bk01
 5      131kB   262kB   131kB                bk02
 6      262kB   524kB   262kB                bk03
 7      524kB   1049kB  524kB                bk04
 8      1049kB  1573kB  524kB                keystore
 9      1573kB  2097kB  524kB                frp
10      2097kB  4194kB  2097kB               countrycode
11      4194kB  8389kB  4194kB               misc
12      8389kB  12.6MB  4194kB               vm-data
13      12.6MB  16.8MB  4194kB               bk06
14      16.8MB  25.2MB  8389kB               logfs
15      25.2MB  33.6MB  8389kB               ffu
16      33.6MB  50.3MB  16.8MB               oops
17      50.3MB  67.1MB  16.8MB               devinfo
18      67.1MB  83.9MB  16.8MB               oem_misc1
19      83.9MB  101MB   16.8MB  ext4         metadata
20      101MB   134MB   32.9MB               bk08
21      134MB   168MB   34.2MB               splash
22      168MB   201MB   33.6MB               bk09
23      201MB   9328MB  9127MB               super
24      9328MB  9328MB  131kB                vbmeta_system_a
25      9328MB  9328MB  131kB                vbmeta_system_b
26      9328MB  9396MB  67.1MB               logdump
27      9396MB  9530MB  134MB                minidump
28      9530MB  9664MB  134MB                rawdump
29      9664MB  10.7GB  1074MB  ext4         cust
30      10.7GB  10.9GB  134MB   ext4         rescue
```

Note the end of the last partition in the above list, 10.9GB, this number will be used as the start of the new `userdata` partition, followed by the end of the partition. Let say that we want to make an approx 40GB `userdata` partition using the following command:&#x20;

```bash
(parted) mkpart userdata   10.9GB 50GB
mkpart userdata   10.9GB 50GB    #If you added "ext4" behind the "userdata" text, your data will be erased when you reboot. 
```

Between `userdata` and 10.9GB are 3 spaces, one of them replace the partition type flag, it is important to use 3 spaces at this step.

Run print again to see the results.

```bash
(parted) print
print
Model: SAMSUNG KLUDG4UHDC-B0E1 (scsi)
Disk /dev/block/sda: 126GB
Sector size (logical/physical): 4096B/4096B
Partition Table: gpt
Disk Flags:
Number  Start   End     Size    File system  Name             Flags
 1      24.6kB  32.8kB  8192B                switch
 2      32.8kB  65.5kB  32.8kB               ssd
 3      65.5kB  98.3kB  32.8kB               dbg
 4      98.3kB  131kB   32.8kB               bk01
 5      131kB   262kB   131kB                bk02
 6      262kB   524kB   262kB                bk03
 7      524kB   1049kB  524kB                bk04
 8      1049kB  1573kB  524kB                keystore
 9      1573kB  2097kB  524kB                frp
10      2097kB  4194kB  2097kB               countrycode
11      4194kB  8389kB  4194kB               misc
12      8389kB  12.6MB  4194kB               vm-data
13      12.6MB  16.8MB  4194kB               bk06
14      16.8MB  25.2MB  8389kB               logfs
15      25.2MB  33.6MB  8389kB               ffu
16      33.6MB  50.3MB  16.8MB               oops
17      50.3MB  67.1MB  16.8MB               devinfo
18      67.1MB  83.9MB  16.8MB               oem_misc1
19      83.9MB  101MB   16.8MB  ext4         metadata
20      101MB   134MB   32.9MB               bk08
21      134MB   168MB   34.2MB               splash
22      168MB   201MB   33.6MB               bk09
23      201MB   9328MB  9127MB               super
24      9328MB  9328MB  131kB                vbmeta_system_a
25      9328MB  9328MB  131kB                vbmeta_system_b
26      9328MB  9396MB  67.1MB               logdump
27      9396MB  9530MB  134MB                minidump
28      9530MB  9664MB  134MB                rawdump
29      9664MB  10.7GB  1074MB  ext4         cust
30      10.7GB  10.9GB  134MB   ext4         rescue
31      10.9GB  50.0GB  39.1GB               userdata
```

Exit the parted tool finally.

Now `userdata` resizing is done. Restart your tablet to apply changes.

{% hint style="info" %}
Here comes a fun thing:

Android stores your data inside the `userdata` partition. When you reset your Android device to its factory default settings, your data will be erased. This is because factory reset is equal to format the `userdata` partition using ext4 filesystem. However, if you delete the `userdata` partition and recreate it without formating, your data will be saved after a reboot. Probably because Android can fix this problem automatically.
{% endhint %}



{% hint style="danger" %}
```
If you added "ext4" behind the "userdata" text, your data will be erased when you reboot. 
```
{% endhint %}

### Assign new EFI partition

Follow [this chapter](enable-uefi-on-any-linux-distributions.md#assign-an-esp-partition) to assign EFI partition.

### Install new system

Enter fastboot mode and repeat the [previous steps](ubuntu-installation-guide-new-method.md#partition-the-ufs).

We'll use the free space for our Ubuntu installtion:

```bash
(parted) mkpart pmos ext4 50.0GB 126GB
mkpart pmos ext4 50.0GB 126GB
```

The output will look like this:

```bash
(parted) print free
print free
Model: SAMSUNG KLUDG4UHDC-B0E1 (scsi)
Disk /dev/block/sda: 126GB
Sector size (logical/physical): 4096B/4096B
Partition Table: gpt
Disk Flags:
Number  Start   End     Size    File system  Name             Flags
        12.3kB  24.6kB  12.3kB  Free Space
 1      24.6kB  32.8kB  8192B                switch
 2      32.8kB  65.5kB  32.8kB               ssd
 3      65.5kB  98.3kB  32.8kB               dbg
 4      98.3kB  131kB   32.8kB               bk01
 5      131kB   262kB   131kB                bk02
 6      262kB   524kB   262kB                bk03
 7      524kB   1049kB  524kB                bk04
 8      1049kB  1573kB  524kB                keystore
 9      1573kB  2097kB  524kB                frp
10      2097kB  4194kB  2097kB               countrycode
11      4194kB  8389kB  4194kB               misc
12      8389kB  12.6MB  4194kB               vm-data
13      12.6MB  16.8MB  4194kB               bk06
14      16.8MB  25.2MB  8389kB               logfs
15      25.2MB  33.6MB  8389kB               ffu
16      33.6MB  50.3MB  16.8MB               oops
17      50.3MB  67.1MB  16.8MB               devinfo
18      67.1MB  83.9MB  16.8MB               oem_misc1
19      83.9MB  101MB   16.8MB  ext4         metadata
20      101MB   134MB   32.9MB               bk08
21      134MB   168MB   34.2MB               splash
22      168MB   201MB   33.6MB               bk09
23      201MB   9328MB  9127MB               super
24      9328MB  9328MB  131kB                vbmeta_system_a
25      9328MB  9328MB  131kB                vbmeta_system_b
26      9328MB  9396MB  67.1MB               logdump
27      9396MB  9530MB  134MB                minidump
28      9530MB  9664MB  134MB                rawdump
29      9664MB  10.7GB  1074MB  ext4         cust
30      10.7GB  10.9GB  134MB   ext4         rescue
        10.9GB  10.9GB  786kB   Free Space
31      10.9GB  50.0GB  39.1GB               userdata
32      50.0GB  126.0GB  76.0GB  ext4         pmos
        
```

Exit the parted tool and reboot once again.

Now we need to check which slot Android is installed.

{% hint style="info" %}
**NOTE:** The concept of "slot"  probably unfamiliar for you, since it was included in Android 10 as a feature, which called "**Dynamic Partitions**". Dynamic partitions are a userspace partitioning system for Android. About its further infomations, [located here](https://source.android.com/docs/core/ota/dynamic_partitions/implement).
{% endhint %}

You can check this infomation via `fastboot` . Command is listed below:

```bash
fastboot getvar current-slot
current-slot: b
Finished. Total time: 0.004s
```

Force select Slot A as active slot:

```bash
fastboot set_active a
Setting current slot to 'a'                        OKAY [  0.049s]
Finished. Total time: 0.064s
```

Now, we are ready to flash the system image. But before we start, we need to disable Android Verified Boot (AVB) feature, otherwise it will prevent booting Ubuntu system.



{% hint style="info" %}
AVB is implementation of verified boot process, current version (since Android 8 Oreo) is called AVB 2.0. Verified boot is a process of assuring the end user of the integrity of the software running on a device. It typically starts with a read-only portion of the device firmware which loads code and executes it only after cryptographically verifying that the code is authentic. It also helps in implementing rollback protection.

&#x20;More information and technical details, [located here.](https://wiki.postmarketos.org/wiki/Android_Verified_Boot_\(AVB\))
{% endhint %}

Flash the `vbmeta` with `vbmeta_disabled.img` to disable this feature:

```bash
fastboot flash vbmeta_a vbmeta_disabled.img
Sending 'vbmeta_a' (4 KB)                          OKAY [  0.007s]
Writing 'vbmeta_a'                                 OKAY [  0.003s]
Finished. Total time: 0.048s
```

Then, flash `rootfs` images:

```
fastboot flash pmos /path/to/rootfs.img
```

## Chrooting new system

{% hint style="info" %}
NOTE: These steps can be done on a regular Android system, but you need to get your device **ROOTED**. Either Magisk or KernelSU is OK.
{% endhint %}

To finish installing our system, we need to use chroot container to continue our setup.

{% hint style="info" %}
According to Wikipedia and Arch Wiki, a [chroot](https://en.wikipedia.org/wiki/Chroot) is an operation that changes the apparent root directory for the current running process and their children. A program that is run in such a modified environment cannot access files and commands outside that environmental directory tree. This modified environment is called a _chroot jail_.
{% endhint %}

Download Orangefox or TWRP recovery first:\
[https://drive.google.com/file/d/1gCNtoDMNCAmMR61xegvCC3mxv28gMJbi/view?usp=drive\_link](https://drive.google.com/file/d/1gCNtoDMNCAmMR61xegvCC3mxv28gMJbi/view?usp=drive_link)

Now, let's boot recovery image temporally:

```bash
fastboot boot /path/to/recovery.img
```

Once the recovery is booted, you are ready to setup a new chroot container. Go to "Advanced" tab, tap "Terminal" to enter terminal, and you are ready to go.

### Prepare new root location

The _chroot_ target should be a directory which contains a file system hierarchy.

In Android recovery, this directory would be `/mnt`. However, on Android, the `/mnt` directory is occupied by other several important directories, so you'll need to create another empty folder for chrooting.

Run `lsblk` and note the partition layout of your installation. It will be usually something like `/dev/sd`_`XY`_.

Mount the file system:

```
# mount /dev/sdXY /mnt
```

If you have an [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition) and need to make changes in it (e.g. updating the [vmlinuz](https://wiki.archlinux.org/title/Vmlinuz) or [initramfs](https://wiki.archlinux.org/title/Initramfs) images):

```
# mount /dev/sdXZ /mnt/efi
```

Then, continue on your recovery ( or Android ) session.&#x20;

### Usage

First, let's mount the temporary API Filesystems:

```
# cd /path/to/new/root
# mount -t proc /proc proc/
# mount -t sysfs /sys sys/
# mount --rbind /dev dev/
```

{% hint style="danger" %}
**Warning:** When using `--rbind`, some subdirectories of `dev/` and `sys/` will not be unmountable. Attempting to unmount with `umount -l` in this situation will break your session, requiring a reboot. If possible, use `-o bind` instead.
{% endhint %}

Next, to change root into _`/path/to/new/root`_ using default `sh` shell:

```
chroot /path/to/new/root /bin/bash
```

Then, we need to fix the permission issues to get chroot fully working under Android or Android recovery:

Simply remount root filesystem:

```
mount -o remount,exec /path/to/new/root
```

Finally, chroot into your system:

```
chroot /path/to/new/root /bin/bash
```

Since this method require UEFI firmware to boot your system, you need to compile the UEFI firmware for your tablet. [Follow this guide](enable-uefi-on-any-linux-distributions.md) to do so.

If you have compiled UEFI firmware, you still need to install a bootloader ( e.g. GRUB ) to load the kernel and start the system. On your chroot container, enter this command to install bootloader:

`sudo apt install grub-efi grub2-common efibootmgr`

```
sudo grub-install --target=arm64-efi --boot-directory=/boot    #Replace /boot with your EFI mountpoint.
sudo grub-mkconfig -o /boot/grub/grub.cfg 
```

Then, type `exit` to exit chroot, and type `reboot bootloader` to reboot your tablet.

Now, flash your UEFI firmware and reboot your tablet:

```
fastboot flash boot /path/to/uefi_image
```

Ensure that the paths and commands are correctly adjusted to fit your system configuration. Once the bootloader installation is complete, verify that the UEFI firmware and bootloader are functioning by rebooting the device and selecting the appropriate boot option from the UEFI menu. This will confirm a successful setup and enable your tablet to boot into the new system environment.

[^1]: 
