# Agon-FAQ
Most frequently asked questions (and answers) to using the Agon platform

## Are these documents available offline or in a different format?
No, and there are no immediate plans on doing this to reduce the amount of time spent doing documentation.

## Is the Agon open source?
The official hardware and software components that are part of the Quark firmware are open source, including BBC BASIC for Agon. Please check the license of any third party applications.

## What else do I need to buy?
Okay, so you've ordered your Agon Light, and are wondering what else you will need to purchase.

Minimum:
- A PS/2 compatible keyboard
- A micro USB card

NB:
- The orginal Agon Light requires a PS/2 keyboard, or a USB keyboard that supports the PS/2 protocol with a USB to PS/2 adaptor.
- The Agon Light 2 requires a USB keyboard that supports the PS/2 protocol, or a PS/2 keyboard with a PS/2 to USB adaptor.

## Does AGON support CP/M?
Officially it is not designed to run CP/M out of the box. There are however third-party developers who are porting CP/M to the platform, and the official build will support this.

## Is there a template available for the SD card, to get me started quickly?
Check out the Popup [MOS repository](https://github.com/tomm/popup-mos); click the green 'Code' icon and select 'Download ZIP' from the dropdown menu. Unzip the file and copy over everything from the 'popup-os-main' folder to an empty SD card. The card should have folders like demos/docs/games/mos/utils in the root of the card

Another great distribution is [Agon Mite](https://agonmite.gluonspace.com/) which has a monthly release cycle.

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

If both VDP and MOS are currently still at 1.03, as most Olimex AgonLight2 boards are, follow [this detailed guide](./Updating-Firmware-from103.md) to upgrade it. This is only necessary once; from 1.04 onwards, all upgrades can be done with the flash tool.

If you are running at least VDP and MOS 1.04, just install the [flash tool](https://github.com/envenomator/agon-flash), get the appropriate firmware files (see below for examples) and read the short documentation on the page to upgrade both VDP and MOS in one go. The latest version can be found under the [releases](https://github.com/envenomator/agon-flash/releases) tab. The page also has a link to help you upgrade, if you are running an even older version than 1.03.

[this flowchart](https://github.com/envenomator/agon-flash/blob/master/assets/update_process.png) provides an overview of the overall process, instructions for all version combinations of VDP and MOS and additional links to use, depending on your current version starting points.

## Where can I find the latest Agon firmware(s) to download?
I recommend using the Console8 firmware.
- [Console8 MOS firmware](https://github.com/AgonConsole8/agon-mos)
- [Console8 VDP firmware](https://github.com/AgonConsole8/agon-vdp/releases)

## Which firmware do you currently recommend using on my Agon?
We recommend using the Console8 firmware, as most development currently takes place there while staying compatible with previously released software. It's important to mention that the latest, most interesting games, make use of features that are only available in the Console8 (VDP) firmware, so make sure to use that.

## Can I switch between the Quark firmware and Console8 firmware?
Yes, either way.

## What is the function of Agon's jumpers and what is their recommended position?
### The ESP jumper
All Agon variants have a jumper location on the PCB marked 'ESP-PROG' or 'ESP-PROG1'. For programming the VDP over the USB interface, this needs to be **CLOSED**, i.e. both pins should be connected using the removable jumper. It is present on the PCB, to provide the option to <em>prohibit</em> the ESP32 to go into programming mode on reset, likely due to the way the USB serial driver behaves on your PC. If this happens to you occasionally and it looks like the Agon doesn't boot on a reset, it may be just waiting for an external programming session that never happens, spewing messages over the USB serial; in that case remove the jumper or leave it open, but remember to close it again before programming it externally over USB. The position of this jumper is ignored when programming the VDP using the flash tool from the MOS prompt.
I recommend to leave this jumper in the **CLOSED** position, unless you experience problems resetting your Agon.

### The UART jumper
All Agon variants have a jumper to either enable, or prohibit the UART between the eZ80 and ESP32. During normal operation of the Agon, both ICs should be able to 'talk' to each other; this jumper was provided in case communication between them would somehow cause issues during programming either IC. AFAIK there haven't been cases to date where this proved necessary, but the option is at least there. If your Agon doesn't do anything after you have received it from your vendor, you may check this jumper's position. Cases have been reported of boards not working due to an incorrect factory jumper setting.

- The AgonLight/AgonLight2/AgonLight Origins edition have this jumper labeled as 'UART-DIS' or 'UART-DIS1', which should be left **OPEN** to not disable communications and have a working Agon

- The Agon Console8 has this jumper labeled as 'UART-EN1', which should be **CLOSED** to enable communications and have a working Agon.

### The Buzzer jumper
All Agon variants have a jumper to enable/disable the onboard buzzer, labeled 'BUZ-EN', 'BUZ_EN1' or 'BUZZER'. Leaving the jumper open disables the buzzer, while closing it enables it, whichever is your preference.

## Do I need to buy the Zilog SMART cable to update my Agon?
No, not unless you will be developing your own Agon MOS firmware AND need to be comfortably sure you can recover it under ALL circumstances. And even then you may very well get by using one of the community-provided options. People in the community owning SMART cables are seldomly using them nowadays.

Again, for regular update purposes, the Zilog SMART cable is unnecessary and a waste of money. If it's bricking you're worried about; there is a [simple solution](https://github.com/envenomator/agon-vdpflash) requiring just a few dupont cables and a clear step-by-step plan to recover your Agon. In any case, you can at least buy a new Agon for the price of the Zilog SMART cable. Ease your worries.

## I still would like to buy a Zilog SMART cable. Which one should I get?
The Agon-compatible Zilog product numbers are:
- ZUS**BSC**00100ZACG (discontinued)
- ZUS**BAS**C0200ZACG (current; requires v5.3.5 or later of Zilog’s ZDS -II IDE)

**ATTENTION:** the cable with product number ZUS**BES**C0200ZACG is **NOT** suitable for Agon!

## Can I go back to an earlier firmware version / downgrade?
Absolutely (and this has been tested), but cecause literally ANY version can be flashed, please be mindful going back to older versions. The Pleistocene may not have a wall-outlet to charge your time travel device.

Do yourselves a big favor, and stay away from anything below Quark 1.03, or that Zilog ZDS cable I mentioned you shouldn't buy, might suddenly start get interesting again.

## Do I understand correctly, that the Console8 firmware(s) can be used with any Agon platform?
Yes, it can be used with any Agon platform.

## How should I connect the Agon to my PC for programming the VDP?
To program an older VDP (1.03 or earlier) to a newer version, you need to connect the Agon using a USB (data) cable. Some very cheap USB cables may only have wires for powering/charging a USB device, without any wired datalines. Fortunately these are becoming scarce nowadays, but if all else fails, you might be using the wrong cable (or have the incorrect jumper setting; see above)

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
Other available layouts are available from MOS 1.03+, [documented here](https://github.com/AgonConsole8/agon-docs/blob/main/MOS.md#keyboard-layout)

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
[Indeed there is](https://github.com/tomm/fab-agon-emulator)

## What is the difference between the Agon Light and the Olimex Agon Light 2?
The main differences are:

- The keyboard connector is USB, yet still requires a keyboard that supports the PS/2 protocol
- LIPO battery charging circuit
- UEXT connector
- USB C power connector
- Plastic boxed 34-pin GPIO connector 

And there are some minor revisions to discrete components on the board. Other than that, it is functionally identical to the original Agon Light design.

## I'm having issues with some video modes
Historically, this has been a problem when the VDP firmware was built using the Arduino IDE.  You are strongly advised to use PlatformIO to build the VDP firmware, and to update to the latest versions; in doing so you should no longer have any issues with screen modes.  PlatformIO takes care of all the dependencies and settings and thus is much easier to use than the Arduino IDE.  The Console8 VDP firmware no longer supports being built using the Arduino IDE and instead is built using PlatformIO to avoid these kind of issues.

If you wish to still use the Arduino IDE and are building the Quark variant of the VDP firmware, you will need to make sure you have PSRAM enabled in the [Arduino IDE settings](https://github.com/breakintoprogram/agon-vdp#arduino-ide-settings) when you compile and transfer the VDP code.

## I'd like to start programming BBC Basic - where might I start?
Have a look at [this](https://oldpatientsea.github.io/agon-bbc-basic-manual/0.1/bbc1.html) excellent guide.
We also provide some [documentation](./BBC-BASIC-for-Agon.md) at our community site.

## Are there any simple BBC BASIC examples?
Your AGON may have a 'tests' folder with some BBC BASIC examples; if it doesn't, they can be [found here](https://github.com/breakintoprogram/agon-bbc-basic/tree/main/tests).

## Is BASIC the only programming language available?
The AGON comes pre-shipped with BASIC as it is a good language to start coding with, but you are not limited to it. The following languages are supported:

- Forth
- Assembly Language (either via BBC Basic inline assembler or external assembler)
- C (Cross-compiled from a PC, via ZDS tools or AgDev)

## I'd like to start assembly programming for the Agon platform - where might I start?
We would recommend watching and learning these [excellent tutorials](https://www.youtube.com/playlist?list=PL-WZxPxo1iaDnvHoGHXeCdKddYqwPaSUk)

Download the latest version of the [onboard assembler](https://github.com/envenomator/agon-ez80asm/releases). Don't dally using the one that came with your uSD card package, things move quickly in this space - use the latest.

## I'd like to start C/C++ programming for the Agon platform - where might I start?
Assuming you already know some C/C++, start downloading the best [C/C++ cross-development toolchain](https://github.com/pcawte/AgDev) there currently is, and look at some online examples from tools/games.

## I'd like to start Forth programming for the Agon platform - where might I start?
There is an excellent Forth implementation available [here](https://github.com/lennart-benschop/agon-forth)

## Is there some in-depth API documentation available for me to look at?
The community is building out [at set of markdown documents](https://github.com/AgonConsole8/agon-docs) to detail several aspects. These are extremely useful to a programmer.

## Is there a documented memory map available to me as a programmer?
A memory map can be found [here](https://github.com/envenomator/Agon/blob/master/Memory%20map.png)

## I'd like to read all details about Agon's ez80 microcontroller
Buckle up!

- [eZ80F92 datasheet](https://www.zilog.com/docs/ez80acclaim/ps0153.pdf) - details regarding the eZ80F92 microcontroller peripherals like I2C, SPI, Flash memory, debug interface, timers
- [eZ80 CPU datasheet](https://www.zilog.com/docs/um0077.pdf) - details regarding the ez80 CPU itself, cpu architecture, registers, memory mode, interrupts and instruction set
- [Application note AN033301-0711](https://www.zilog.com/docs/appnotes/an0333.pdf) - details of the ZDS Application Binary Interface (ABI), how to call C functions from Assembly and vice versa. Also applicable to the AgDev environment which is compliant to the ABI.

## Can you code feature {x}?
All suggestions are welcome, though the developers will be concentrating on key features. If we think your idea has legs, we'll add it to the pile.

## Why is feature {x} not documented anywhere?
You might have found an opportunity to make a valuable contribution to the Agon community. Take a look at the community [documentation](https://github.com/AgonConsole8/agon-docs) and create a pull request.