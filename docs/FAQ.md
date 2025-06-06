---
hide:
  # Hide the TOC on the right hand side as we are overriding it on this page only
  - toc
---
# Agon Frequently Asked Questions

The following questions (and answers) on using the Agon platform are below:

[TOC]

## Are these community documents available offline or in a different format?

No, and there are no immediate plans on doing this to reduce the amount of time spent doing documentation.

This documentation however is all [open source](https://github.com/AgonPlatform/agon-docs), and written in [Markdown](https://www.markdownguide.org) format.  Pull requests suggesting changes to the documentation are always welcome.

## Is the Agon open source?

Yes.  All of the firmware for the Agon platform is open source, for both [MOS](https://github.com/AgonPlatform/agon-mos) and the [VDP](https://github.com/AgonPlatform/agon-mos), and also including the various versions of BBC BASIC for the Agon ([BASIC 4 for Z80](https://github.com/breakintoprogram/agon-bbc-basic), [BASIC 4 for eZ80](https://github.com/breakintoprogram/agon-bbc-basic-adl) and [BASIC V for Z80](https://github.com/breakintoprogram/agon-bbc-basic-v)).

[Bernado Kastrup](https://www.thebyteattic.com), the original hardware designer of the Agon Light, released the full schematics for the [Agon light](https://github.com/TheByteAttic/AgonLight) and the [Agon light ORIGINS edition](https://github.com/TheByteAttic/AgonORIGINS) as open source.  He also designed Heber Ltd.'s [Agon Console8](https://heber.co.uk/agon-console8/), available from the Heber and Retro Collective [shop](https://shop.heber.co.uk/agon-console8/).

The [Olimex AgonLight2](https://www.olimex.com/Products/Retro-Computers/AgonLight2/open-source-hardware) is based on the original Agon Light design, which is also an open source design.

## What else do I need to buy?

Okay, so you've ordered your Agon Light, and are wondering what else you will need to purchase.

As a minimum, you also need the following extra hardware:

- A PS/2 compatible keyboard
- A micro USB card

!!! note

    - The original Agon Light requires a PS/2 keyboard, or a USB keyboard that supports the PS/2 protocol with a USB to PS/2 adaptor.
    - The Agon Light 2 and Agon Console8 require a USB keyboard that supports the PS/2 protocol, or a PS/2 keyboard with a PS/2 to USB adaptor.

## Does Agon support CP/M?

Officially it is not designed to run CP/M out of the box. There are however third-party developers who have ported CP/M to the platform, and the official build supports this.

There is also available a [CP/M compatibility layer](https://github.com/nihirash/zinc), allowing some CP/M software to run directly from the MOS command line.

## Is there a template available for the SD card, to get me started quickly?

Check out the Popup [MOS repository](https://github.com/tomm/popup-mos); click the green 'Code' icon and select 'Download ZIP' from the dropdown menu. Unzip the file and copy over everything from the 'popup-os-main' folder to an empty SD card. The card should have folders like demos/docs/games/mos/utils in the root of the card

Another great distribution is [Agon Mite](https://www.agonmite.com) which aims to have a monthly release cycle.

These repositories are a volunteer effort, periodically collecting the latest versions of software out there. If you need a more recent version of any package, because sometimes development goes pretty fast; please download it individually to your SD card.

## What kind of SD card should I use?

Any class-10 micro SD card of a decent quality will do fine

## What size SD card should I buy?

You should aim for a maximum card size of 32GB. Larger cards are supported (using FAT32), but you may run into issues partitioning and formatting them.

## How should I format the SD card initially?

Format the card using a FAT32 layout

## Will my Agon boot directly into BBC BASIC or will I have to download the language onto an SD card?

Most distributions contain both the BBC BASIC binary (bbcbasic.bin), and an autoexec.txt file at top level of SD card. The latter contains commands for auto loading BBC Basic. If you'd like to disable auto loading BBC BASIC; remove it, or change the content to suit your own requirements.

## How should I power the Agon?

All Agon platforms can be powered using a USB cable to your PC, or a USB wall-charger. The load is relatively light, usually below 200mA with a connected keyboard. Which USB cable you need, depends on the variant of Agon you buy. For example, the Olimex AgonLight2 is powered from a USB-C interface, while the Agon Console8 can be powered using a USB-B interface. The latter can also be powered with 5v DC jack connector.

## How can I update the Agon?

It really depends on the current firmware versions on your Agon. The Agon contains firmware for both the OS (MOS) and graphics unit (VDP).

Full guidance on updating your Agon can be found in [this detailed guide](./Updating-Firmware.md).

## Where can I find the latest Agon firmware(s) to download?

The latest Agon platform firmware releases for MOS and the VDP can be found here:

- [MOS firmware](https://github.com/AgonPlatform/agon-mos/releases/latest)
- [VDP firmware](https://github.com/AgonPlatform/agon-vdp/releases/latest)

## Quark, Console8, or Agon Platform firmware??

The original "official" firmware for the Agon Light was primarily written by [Dean Belfield](http://www.breakintoprogram.co.uk), with help from several other contributors, and was given the name "Quark".

At the time of writing, the latest Quark firmware releases are Quark MOS 1.04 and Quark VDP 1.04, which were released in November 2023.  Dean is a busy man with a full-time job, and a family to look after, so his available time to work on the Agon is limited.  Since those firmware releases, he has concentrated his Agon efforts since then in developing and maintaining versions of BBC BASIC for the Agon.

The Agon Console8 firmware started as a fork of the Quark firmware, made by [Steve Sims](https://www.patreon.com/c/SteveSims).  An AgonConsole8 organisation was set up on GitHub, and hard forks made of the Quark firmware.  There are many reasons why this fork was made which won't be explored here.  The two most important reasons however were to allow for a more rapid release schedule, and to have a wider organisation in place for looking after the firmware, to avoid the "hit by a bus" risk of a single owner.  Initially all changes made to the Console8 firmware were raised as pull requests to be merged back into the Quark firmware, and broadly speaking the Quark 1.04 and Console8 2.0.0 releases were nearly identical.

Owing to the pace of development, it stopped being practical to contribute changes made to the Console8 firmware (as it was then known) back to the Quark firmware early in 2024.

Despite it's name, the Console8 firmware has always been able to be used on all Agon platforms, including the Agon Light, Agon Light 2, and Agon Origins Edition.  Whilst this has always been the case, the "Console8" name has lead to some confusion...

"Agon Platform" is the new name for the Console8 firmware, and also the new name for the organisation.  This rename happened in April 2025.

## Which firmware do you currently recommend using on my Agon?

We recommend using the Agon Platform firmware, as most development currently takes place there while staying compatible with previously released software. It's important to mention that the latest, most interesting games, make use of features that are only available in the Agon Platform (VDP) firmware, so make sure to use that.

## Can I switch between the Quark firmware, Console8, and Agon Platform firmware?

Yes, either way.

## What is the function of Agon's jumpers and what is their recommended position?

### The ESP jumper

All Agon variants have a jumper location on the PCB marked 'ESP-PROG' or 'ESP-PROG1'. For programming the VDP over the USB interface, this needs to be **CLOSED**, i.e. both pins should be connected using the removable jumper. It is present on the PCB, to provide the option to *prohibit* the ESP32 to go into programming mode on reset, likely due to the way the USB serial driver behaves on your PC. If this happens to you occasionally and it looks like the Agon doesn't boot on a reset, it may be just waiting for an external programming session that never happens, spewing messages over the USB serial; in that case remove the jumper or leave it open, but remember to close it again before programming it externally over USB. The position of this jumper is ignored when programming the VDP using the flash tool from the MOS prompt.
I recommend to leave this jumper in the **CLOSED** position, unless you experience problems resetting your Agon.

### The UART jumper

All Agon variants have a jumper to either enable, or prohibit the UART between the eZ80 and ESP32. During normal operation of the Agon, both ICs should be able to 'talk' to each other; this jumper was provided in case communication between them would somehow cause issues during programming either IC. AFAIK there haven't been cases to date where this proved necessary, but the option is at least there. If your Agon doesn't do anything after you have received it from your vendor, you may check this jumper's position. Cases have been reported of boards not working due to an incorrect factory jumper setting.

- The AgonLight/AgonLight2/AgonLight Origins edition have this jumper labeled as 'UART-DIS' or 'UART-DIS1', which should be left **OPEN** to not disable communications and have a working Agon
- The Agon Console8 has this jumper labeled as 'UART-EN1', which should be **CLOSED** to enable communications and have a working Agon.

### The Buzzer jumper

All Agon variants have a jumper to enable/disable the onboard buzzer, labeled 'BUZ-EN', 'BUZ_EN1' or 'BUZZER'. Leaving the jumper open disables the buzzer, while closing it enables it, whichever is your preference.

## Do I need to buy the Zilog SMART cable to update my Agon?

No, not unless you will be developing your own Agon MOS firmware AND need to be comfortably sure you can recover it under ALL circumstances. And even then you may very well get by using one of the community-provided options. People in the community owning SMART cables are seldomly using them nowadays.

As an example of this, most of the development of the Agon Platform MOS firmware has been done without the use of the Zilog SMART cable.

Again, for regular update purposes, the Zilog SMART cable is unnecessary and a waste of money. If it's bricking you're worried about; there is a [simple solution](https://github.com/envenomator/agon-recovery) requiring just a few dupont cables and a clear step-by-step plan to recover your Agon. In any case, you can at least buy a new Agon for the price of the Zilog SMART cable. Ease your worries.

## I still would like to buy a Zilog SMART cable. Which one should I get?

The Agon-compatible Zilog product numbers are:

- ZUS**BSC**00100ZACG (discontinued)
- ZUS**BAS**C0200ZACG (current; requires v5.3.5 or later of Zilog’s ZDS -II IDE)

!!! danger "Attention"

    The cable with product number ZUS**BES**C0200ZACG is **NOT** suitable for Agon!

## Can I go back to an earlier firmware version / downgrade?

Absolutely (and this has been tested), but because literally ANY version can be flashed, please be mindful going back to older versions. The Pleistocene may not have a wall-outlet to charge your time travel device.

Do yourselves a big favor, and stay away from anything below Quark 1.03, or that Zilog ZDS cable I mentioned you shouldn't buy, might suddenly start get interesting again.

## I don't have a Console8, can I still use the Console8 firmware?

Yes, the Console8 firmware can be used on any existing Agon platform computer.  This includes the original Agon Light, the Olimex Agon Light 2, the Agon Origins Edition, as well as the Agon Console8.

There are no features in the firmware that are exclusive to the Agon Console8.  It is possible to add a PS/2 mouse port and two joystick ports to Agon Light machines in a manner that is completely compatible with the Console8 firmware.

To help reduce this confusion, the "Console8" name has been dropped for firmware releases in favour of "Agon Platform" in April 2025.  The Console8 firmware is now known as the Agon Platform firmware.

## How should I connect the Agon to my PC for programming the VDP?

To program an older VDP (1.03 or earlier) to a newer version, you need to connect the Agon using a USB (data) cable. Some very cheap USB cables may only have wires for powering/charging a USB device, without any wired data lines. Fortunately these are becoming scarce nowadays, but if all else fails, you might be using the wrong cable (or have the incorrect jumper setting; see above)

The Agon will present a virtual serial port (COM) on your PC, that communicates directly with the ESP32 graphics chip (VDP).

## How can I find out the COM port on my PC, to connect to the Agon?

It depends on the OS you have. In Linux, when you insert your USB cable and type 'dmesg | grep tty', it should show.

In Windows, it should show up in device manager, or in powershell type '[System.IO.Ports.SerialPort]::getportnames()'.

In Mac OS, you can type 'ls /dev/tty.*' in Terminal to list all com ports currently attached.

## Why is Agon model XXX delivered with out-of-date firmware?

A legitimate question that you need to ask the respective vendor of your Agon.

## What keyboard do I need for my Agon?

You need a keyboard that supports the PS/2 protocol, regardless of the physical connection that it has. Some Agon variants have a PS/2 interface, so any keyboard with a PS/2 connector should be fine.
However, a number of Agon variants use a USB connector for their keyboard interface; for these you either need a USB keyboard that still supports the PS/2 protocol in backward compatibility mode (and it is usually hard to find out if it does), or a regular PS/2 keyboard with a PASSIVE PS/2-to-USB connector. An example for the latter, which is known to work well [is this one](https://www.amazon.com/Female-Adapter-Converter-Connector-Mouse/dp/B0BHRPVJZD/ref=sr_1_30?crid=3SG167E0EZITQ&keywords=Passive%20ps2%20adapter&qid=1687116944&sprefix=passive%20ps2%20adapter%2Caps%2C169&sr=8-30&fbclid=IwAR14ReD4S5cFHeTfE6eF03ec5_bbYQ7_NgplqCEzzluxCJY_9oq0uiOHcIQ). This is a pass-through connector, that just rewires the PS/2 signals to the USB connector. Perfect for the Agon.

A PS/2-to-USB connector cannot make use of active logic and/or microcontrollers inside the connector. If the connector is dirt-cheap, it's usually passive, but even active converters can be offered relatively cheap.

If a PS/2->USB converter has both a PS/2 keyboard and PS/2 mouse cable, a thick USB connector and/or a 'bulge' in the middle of the cable AND it doesn't specifically mention the term **PASSIVE**, it's a dead giveaway to be an active (non-supported) converter. Don't buy these: they scan/convert the PS/2 signals to a USB HID protocol that the Agon doesn't support. Examples to avoid [1](https://www.amazon.com/UCEC-Mouse-Keyboard-Converter-Cable/dp/B00X9QX9MS/ref=sr_1_9?crid=114AET38YBVMF&dib=eyJ2IjoiMSJ9.sD5kOGVjIYYaePDdcac3Zkqa5L5aKdNpzLx-i6lyOI0v9eQEk79fF3DXXJqAwJ0T4H1QiNsePhvnsBHdOqgM4fWvGCJTsZ5bv953oMiQ95lCEOP6zjSqoVS4mshWQjgRNzlGqrHnCJBeCzSqMAiWf3sBl5z_9TXHB1PmOUv4TPAySoDDtXWxcdSOlIsIve9OhejtVS5y2WX4uJJXoE1ZxDW7udIoZ8f1kTpeDt0I5qg.t5T8a8o73hLFRS0P3D_EW0e5KcDVaS7hnT1WdQ6irb4&dib_tag=se&keywords=ps2+to+usb&qid=1710795179&sprefix=ps2+to+us%2Caps%2C178&sr=8-9) and [2](https://www.amazon.com/LEIHONG-Active-Adapter-Keyboard-Converter/dp/B088LPKRLD/ref=sr_1_15?crid=114AET38YBVMF&dib=eyJ2IjoiMSJ9.sD5kOGVjIYYaePDdcac3Zkqa5L5aKdNpzLx-i6lyOI0v9eQEk79fF3DXXJqAwJ0T4H1QiNsePhvnsBHdOqgM4fWvGCJTsZ5bv953oMiQ95lCEOP6zjSqoVS4mshWQjgRNzlGqrHnCJBeCzSqMAiWf3sBl5z_9TXHB1PmOUv4TPAySoDDtXWxcdSOlIsIve9OhejtVS5y2WX4uJJXoE1ZxDW7udIoZ8f1kTpeDt0I5qg.t5T8a8o73hLFRS0P3D_EW0e5KcDVaS7hnT1WdQ6irb4&dib_tag=se&keywords=ps2%2Bto%2Busb&qid=1710795253&sprefix=ps2%2Bto%2Bus%2Caps%2C178&sr=8-15&th=1)

## Can you recommend a PS/2 keyboard?

Sure, there's a crowd-sourced spreadsheet [here](https://docs.google.com/spreadsheets/d/1-6_sz6l-vJW5rFg3M0Y6bwC0hmFS7U6PPNjIZ9plrM8/edit?fbclid=IwAR0SBuM-oCywGE7Km6PupIWpiKnQNXIHQ2hD7iSDo5T7b_LTHXX0JNEe3Fw#gid=0) with a list of known working keyboards.

If you have one, and it's not on the list, then please feel free to add an entry.

## How do I configure a different keyboard layout?

You can use the following command from the MOS command line, from BBC BASIC (using the *), or loaded on boot/reset using a line in the autoexec.txt file on your SD card:

    *SET KEYBOARD n

The keyboard layout for UK = 0, US = 1

The keyboard layouts your Agon can support depends on the firmware versions you are using; at the time of writing 17 different layouts are supported.  More information about available layouts is [documented here](mos/Star-Commands.md#keyboard-layout)

## How do I exit to the MOS command line, from within BBC Basic?

Use the following command:

    *BYE

## Where can I find the user/hardware manual for my Agon edition?

Check out these links for your particular Agon edition:

- [Byte Attic AgonLight R1.0](https://github.com/TheByteAttic/AgonLight/blob/main/Agon%20light%20R1.0%20Manual.pdf)
- [Byte Attic Agon Origins edition](https://github.com/TheByteAttic/AgonORIGINS/blob/main/Agon%20light%20ORIGINS%20Manual.pdf)
- [Olimex AgonLight2](https://github.com/OLIMEX/AgonLight2/blob/main/DOCUMENTATION/AgonLight2-user-manual.pdf)
- [Agon Console8](https://heber.co.uk/agon-cosole8/)

## Is there a community list of Agon software?

We're listing a few here, though there will certainly be others:

- [Sabotrax/agon-software](https://github.com/sabotrax/agon-software)
- [The Byte Attic - Agon overview](https://www.thebyteattic.com/p/agon.html)

## Is there an Emulator available?

Indeed there is, the [Fab Agon Emulator](https://github.com/tomm/fab-agon-emulator). It is available for Windows, Linux and Mac (or you can [compile](https://github.com/tomm/fab-agon-emulator/blob/main/docs/compiling.md) your own).

## What is the difference between the Agon Light and the Olimex Agon Light 2?

The main differences are:

- The keyboard connector is USB, yet still requires a keyboard that supports the PS/2 protocol
- LIPO battery charging circuit
- UEXT connector
- USB C power connector
- Plastic boxed 34-pin GPIO connector

And there are some minor revisions to discrete components on the board. Other than that, it is functionally identical to the original Agon Light design.

## I'm having issues with some video modes

Historically, this has been a problem when the VDP firmware was built using the Arduino IDE.  You are strongly advised to use PlatformIO to build the VDP firmware, and to update to the latest versions; in doing so you should no longer have any issues with screen modes.  PlatformIO takes care of all the dependencies and settings and thus is much easier to use than the Arduino IDE.  The Agon Console8 VDP firmware (now known as the Agon Platform VDP firmware) no longer supports being built using the Arduino IDE and instead is built using PlatformIO to avoid these kind of issues.

If you wish to still use the Arduino IDE and are building the Quark variant of the VDP firmware, you will need to make sure you have PSRAM enabled in the [Arduino IDE settings](https://github.com/breakintoprogram/agon-vdp#arduino-ide-settings) when you compile and transfer the VDP code.

Finally there are a few screen modes that the Agon supports that a limited number of screens might not support.  If you are having issues with a particular screen mode, try using a different screen mode to see if that resolves the issue.  Remember that you can always press the reset button.

## I'd like to start programming BBC Basic - where might I start?

Have a look at [this](https://oldpatientsea.github.io/agon-bbc-basic-manual/0.1/bbc1.html) excellent guide.
We also provide some [documentation](./BBC-BASIC-for-Agon.md) at our community site.

## Are there any simple BBC BASIC examples?

Your AGON may have a 'tests' folder with some BBC BASIC examples; if it doesn't, they can be [found here](https://github.com/breakintoprogram/agon-bbc-basic/tree/main/tests).

## Is BASIC the only programming language available?

The AGON comes pre-shipped with BASIC as it is a good language to start coding with, but you are not limited to it. The following languages are supported:

- Forth
- Assembly Language (either via BBC Basic inline assembler or external assembler)
- C (Cross-compiled from a PC, via ZDS tools, Agondev, or AgDev)
- C++ (Cross-compiled from a PC or Mac, via Agondev or AgDev)

## I'd like to start assembly programming for the Agon platform - where might I start?

We would recommend watching and learning these [excellent tutorials](https://www.youtube.com/playlist?list=PL-WZxPxo1iaDnvHoGHXeCdKddYqwPaSUk)

Download the latest version of the [onboard assembler](https://github.com/envenomator/agon-ez80asm/releases). Don't dally using the one that came with your uSD card package, things move quickly in this space - use the latest.

## I'd like to start C/C++ programming for the Agon platform - where might I start?

Assuming you already know some C/C++, we recommend downloading and using [agondev](https://github.com/AgonPlatform/agondev) as it is the best cross-development toolchain available for the Agon platform.  This is a comparatively new toolchain and is still in active development.

Another option is the [AgDev](https://github.com/pcawte/AgDev), which is an extension of the CEDev toolchain originally written to allow programs to be written for the TI-85 Plus CE and TI-83 Premium CE calculators, which use the same eZ80 processor as the Agon.  There are a number of example programs available in the AgDev repository.

Agondev has a decent degree of compatibility with AgDev, but tends to be slightly more compliant with modern C and C++ standards.

## I'd like to start Forth programming for the Agon platform - where might I start?

There is an excellent Forth implementation available [here](https://github.com/lennart-benschop/agon-forth)

## Is there some in-depth API documentation available for me to look at?

The community is building out [at set of markdown documents](https://github.com/AgonPlatform/agon-docs) to detail several aspects. These are extremely useful to a programmer.

## Is there a documented memory map available to me as a programmer?

A memory map can be found [here](https://github.com/envenomator/Agon/blob/master/Memory%20map.png)

## I'd like to read all details about Agon's ez80 microcontroller

Buckle up!

- [eZ80F92 datasheet](https://www.zilog.com/docs/ez80acclaim/ps0153.pdf) :fontawesome-solid-file-pdf: - details regarding the eZ80F92 microcontroller peripherals like I2C, SPI, Flash memory, debug interface, timers
- [eZ80 CPU datasheet](https://www.zilog.com/docs/um0077.pdf) :fontawesome-solid-file-pdf: - details regarding the ez80 CPU itself, cpu architecture, registers, memory mode, interrupts and instruction set
- [Application note AN033301-0711](https://www.zilog.com/docs/appnotes/an0333.pdf) :fontawesome-solid-file-pdf: - details of the ZDS Application Binary Interface (ABI), how to call C functions from Assembly and vice versa. Also applicable to the AgDev environment which is compliant to the ABI.

## Can you code feature {x}?

All suggestions are welcome, though the developers will be concentrating on key features. If we think your idea has legs, we'll add it to the pile.

## Why is feature {x} not documented anywhere?

You might have found an opportunity to make a valuable contribution to the Agon community. Take a look at the community [documentation](https://github.com/AgonPlatform/agon-docs) and create a pull request.

## I'd like to discuss my project or ideas with like-minded people

Perhaps you have some cool ideas and you want to run them past people, or you want to watch some of the developers talk, or you've followed all the instructions here and broke something and now you'd like to ask around to see if someone else has had the same experience? There is a [Discord server](https://discord.gg/sN3vXru26s) available for Agon & Console 8 community discussions.
