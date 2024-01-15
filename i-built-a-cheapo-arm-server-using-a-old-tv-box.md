---
description: >-
  Having an unused TV box lying around? You might as well want to give it a new
  change to shine.
---

# 📟 I built a cheapo arm server using a old TV box

## Why?



## Prerequisites

* **Tanix TX3 mini**: this one have so many variants mine is A
  * **SoC**: S905W
  * **RAM**: 2GB
  * **ROM** (eMMc): 16GB
* SD card or USB with **at least 8GB** of storage for the installer.
  * _Optional: a big-ass SD card for later use as internal storage, a stick out USB device doesn't fit my aesthetic need._
* Firmware - You might need both in-case your current firmware doesn't allow accessing to bootloader mode like mine. (I'm really much appreciated to these guys for their hard work on the firmware 🙏🏻):
  * OpHub Armbian: [https://github.com/ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)
    * Either Linux 5 or 6 should work just fine.
  * Aidan's ROM: [https://aidanrom.com/project/s905w/](https://aidanrom.com/project/s905w/)&#x20;
    * Besure to pick the correct version for your device.
* Amlogic Burning tool v2.1.7 - this version seems to be less buggy.
* Balena Etcher for creating armbian installation medium.
* Toothpick if you're Asian like me or a SIM ejection tool if you ain't.

## Prepare the installation medium

### Write the firmware

1. Insert SD Card/USB flash drive
2. Launch Belena Etcher
3. Select the firmware
4. Select the destination
5. Flash

### Modify the bootloader

Open the installation medium, check the following files:

1.  `/extlinux/extlinux.conf` should show

    `FDT /dtb/amlogic/meson-gxl-s905w-p281.dtb`
2. Overwrite the uboot file with the uboot.s905x
3. Remove `.template` from the `first_run.template`

Safely eject the medium, it's ready.

## Flash back to Aidan's

## Install armbian

### Boot to the installation medium

This is the hardest part, why? Cause you're alone and don't have anyone to give you a hand on plugging in the power cable and pick the reset button on the device at the same time.

1. Insert the medium
2. Locate the hidden reset button and prepare the toothpick/SIM ecjection tool.
3. Locate the ready the power cable.
4. Plug both of the power cable and press the reset button and hold for a few seconds untill you see something on the screen.

{% hint style="info" %}
If you don't get any signal on boot, don't panic, try to connect to an 1080p monitor.
{% endhint %}

### Install armbian

Pretty straight forward, follow the on-screen installation menu.

## Notes

*   Variants of the box:

    * TX3MINI-XB(P) - Using 9012p wifi chipset

    <figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption><p>credit to <a href="https://www.forum.atvxperience.com/memberlist.php?mode=viewprofile&#x26;u=33455&#x26;sid=7a9b5e03e7caeac27970782b5be115a6">phansyvu</a></p></figcaption></figure>


* A
* A

## References

* [https://i12bretro.github.io/tutorials/0393.html](https://i12bretro.github.io/tutorials/0393.html)
* [https://i12bretro.github.io/tutorials/0316.html](https://i12bretro.github.io/tutorials/0316.html)
*
