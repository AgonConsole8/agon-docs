# Updating the Agon Firmware

When new versions of the Agon firmware are released, you will need to update your Agon machine to take advantage of the new features and bug fixes.  All models of Agon machines to date can run the latest versions of the Agon platform firmware.  This includes the original Agon Light, the Olimex Agon Light 2, the Agon Console8, and the Agon Light Origins Edition.

The Agon firmware is made up of two components:

- The Agon MOS (the operating system, which runs on the eZ80 CPU)
- The VDP (the graphics chip, which runs on the ESP32)

These two components are updated separately.  The exact steps to update your Agon will depend on the versions of the firmware your Agon is currently running.

!!! danger "Attention"

    Before you continue, please note that if your Agon is running a version of MOS before 1.03, or a version of the VDP before 1.04RC3, then the upgrade process is a little different.  It is important that you follow the guidance that can be found [below](#update-old).  Failing to do so may result in your Agon becoming unusable.  [Recovery tools](#recovery) are available to restore your Agon to a working state, but it is best to avoid needing them in the first place.

The primary tool to update the firmware on your Agon is the [Agon firmware update utility](https://github.com/AgonConsole8/agon-flash), otherwise known as "agon-flash".  This tool runs on your Agon, and can be used to update both MOS and the VDP so long as they are already running compatible versions.  The latest version of the this tool can be [downloaded here](https://github.com/AgonConsole8/agon-flash/releases/latest).

Updating the VDP firmware can also be done using the online [Agon VDP installer](#vdp-installer).  Many users find this to be the easiest way to update the VDP firmware.

## How to use the Agon firmware update utility

The Agon firmware update utility is run on your Agon, and needs to be installed onto the SD card you use with your Agon.  That SD card must be formatted as FAT32.

1. Make sure to create a **mos** directory on the microSD card, if it's not already present.
2. Place the [flash.bin](https://github.com/AgonConsole8/agon-flash/releases/latest/download/flash.bin) in the **mos** directory
3. Place the firmware files in the **root** directory of the microSD card:
    - MOS firmware - default filename 'MOS.bin' (download [latest](https://github.com/AgonConsole8/agon-mos/releases/latest/download/MOS.bin) version)
    - VDP firmware - default filename 'firmware.bin' (download [latest](https://github.com/AgonConsole8/agon-vdp/releases/latest/download/firmware.bin) version)

Once you have installed the Agon firmware update utility, and ensured that you have the latest firmware files on your SD card, it can be used to update the firmware on your Agon.

As noted above, to update MOS using this tool, your Agon must be running at least MOS 1.03.  If you are running an older version of MOS, you will need to use the older method of updating MOS.  See [below](#update-old) for more details.

To upgrade MOS, you can use the following command from the MOS command line:

```
*flash mos MOS.bin
```

Updating the VDP using this tool requires a version of the VDP firmware that supports the update process.  This is VDP 1.04RC3 or later.  If you are running an older version of the VDP, you will need to use the older method of updating the VDP.  See [below](#update-old) for more details.

To upgrade the VDP, you can use the following command from the MOS command line:

```
*flash vdp firmware.bin
```

The update process will take a few seconds, and you will be asked to confirm that you wish to flash a new version of your firmare.  Once you press "Y" to confirm the update process will begin, and the Agon will reboot automatically once it is complete.

On rare occasions the Agon may not reboot automatically - you may see the countdown, but then be left on the update screen.  If this happens, just press the "reset" button on the Agon to reboot it, or switch your Agon off and back on.

When asking for confirmation, the tool will display a CRC checksum for the file you are about to install; we no longer publish the checksums for firmware files, as in general we found that most people did not find this useful.  (Older versions of the flash tool would require you to manually enter the CRC number as part of the flash command which users found frustrating as it was easy to make mistakes when typing in the checksum.)  Should you wish to verify the checksum of the firmware you are about to install you can generate a CRC32 checksum for the firmware file on your desktop computer using a tool such as `crc32`.

### Issues with updating from some Agon VDP firmware releases using the Agon firmware update utility

There have been three releases of the Agon VDP firmware that unfortunately contained a bug that prevented the Agon firmware update utility from working correctly.  These releases are:

* VDP 2.8.0
* VDP 2.12.0
* VDP 2.13.0



## How to use the online Agon VDP installer {#vdp-installer}

The online [Agon VDP installer](https://envenomator.github.io/) is a web-based tool that allows you to update the VDP firmware on your Agon without needing to download and install the Agon firmware update utility.

The online Agon VDP installer will work with any Agon machine, running any firmware version.  Please note though that you should not use this tool unless you are running at least MOS 1.03 - if you are running an older version of MOS then it is very important to [update that first](#update-old).

There are two pre-requisits to using this tool.

1. You must be using a web browser that includes "Web Serial" support, which includes Google Chome/Chromium, Microsoft Edge, and Opera
2. Your Agon must be connected to your PC (or Mac) using a USB data cable. The type of USB cable may differ according to the Agon platform you're using:
	- the original Agon Light has a USB-A type connector *
	- the Olimex Agon Light 2 has a USB-C type connector
	- the Agon Console8 has a USB-B type connector
	- the Agon Origins edition has a USB-B type connector

\* The original Agon Light's use of a USB-A connector can cause some problems, as a USB-A to USB-A cable is not a standard cable.  If your PC has a USB-C connector you may be able to use a USB-A to USB-C cable.

Many people power their Agon via the Agon's USB connector.  If you do so by plugging your Agon in to one of your PC's USB ports, so long as your cable is a data cable and not a power-only cable, you should already be able to use the Agon VDP installer.  If you are using a power-only cable, you will need to use a different cable.

Using the online installer is very simple.  Just visit the installer, select the latest version of the VDP firmware, and click on the update button.  Once the update is complete, the Agon will reboot automatically.

## Upgrading from older firmware versions {#update-old}

If your Agon is running older versions of the firmware then you will need to follow a different process to update your Agon.  This is because versions of MOS before 1.03 and the VDP before 1.04RC3 do not support the current Agon firmware update utility.

In general, the simplest way to update your Agon is to follow these two steps:

1. Update MOS to the latest version using the [Agon legacy firmware update utility](https://github.com/AgonConsole8/agon-flashlegacy)
2. Update your VDP firmware to the latest version using the online [Agon VDP installer](https://envenomator.github.io/)

It is very important that you update MOS first, as there was a change in how MOS and the VDP communicated in early firmware versions.  If you try to update the VDP first then you will not be able to run the Agon legacy firmware update utility, and you will not be able to update MOS.

After updating MOS you may find that your Agon appears to not be working properly.  It may fail to start up completely, it might only show the VDP version line, or it may appear to boot but not respond to keyboard input.  This is normal, and is expected behaviour.  Once you have updated the VDP firmware, your Agon will work properly again.

## Other options for updating your VDP firmware

If for some reason you are unable to use the online Agon VDP installer, and you are unable to use the Agon firmware update utility, then there are two remaining options for updating your VDP firmware.  Please note that neither of these options are recommended, and you should only use them as a last resort.

Both of these options require that your Agon is connected to your PC using a USB data cable.

### Using PlatformIO

The Agon VDP firmware can be built from its [source code](https://github.com/AgonConsole8/agon-vdp) using the [PlatformIO IDE](https://platformio.org) extension to [Microsoft Visual Studio Code](https://code.visualstudio.com).  This approach is how the firmware is developed, and built for release.

You will need to download the source code, either by cloning the repository using `git`, or by downloading the source code as a ZIP file and extracting it.  When you open up the source code in Visual Studio Code then from the PlatformIO extension under "Project Tasks > esp32dev > General" you can select "Upload" to build and upload the firmware to your Agon.

If you are not comfortable with using developer tools this is probably not the best option for you.

### Using other tools

Guidance on using other tools can be [found here](https://github.com/envenomator/envenomator.github.io).

In general these tools are not recommended for use by end users, as they are not very user friendly.

## Recovery

Usually updating your Agon firmware will go smoothly, and you will not have any problems.

The process of updating the VDP firmware is designed to be as simple and robust as possible, and is designed to be able to recover from all problems.  The VDP actually keeps two copies of its firmware installed, and so in the rare event that something does go wrong when updating the VDP firmware the VDP simply won't switch to the new firmware.  This means that you can safely update your Agon without worrying.  In the incredibly rare event that this does not work then the [online Agon VDP installer](https://envenomator.github.io/) can be used to install the latest VDP firmware.

It is not possible to make the upgrade process for MOS quite as robust, as it cannot keep two copies of the firmware installed.  There is however an [Agon recovery utility](https://github.com/AgonConsole8/agon-recovery) available.

Full guidance on using the recovery utility is [found here](https://github.com/AgonConsole8/agon-recovery#readme).

The guidance will let you know that if you have an Agon Light, Agon Light 2, or Agon Light Origins Edition, then as well as connecting your Agon to your PC via a USB data cable you will also need two dupont female-to-female cables to make some connections between the ports on your Agon.  The whole process including which connections to make are clearly described.

If you have an Agon Console8 then recovery is a little more complex.  You will need an external ESP32 to perform the recovery.  Most ESP32 devkits will do fine for this purpose (there are lots of options, are widely available, and can be bought for less than $5 on AliExpress), or if you have another Agon Light system available that can also be used as a programmer for the Agon Console8.  The recovery process is a little more complex, but is still well documented.

## Getting help

If you have any problems with updating your Agon firmware, or if you have any questions, please feel free to ask for help in the [Agon & Console8 Community Discord server](https://discord.gg/sN3vXru26s).
