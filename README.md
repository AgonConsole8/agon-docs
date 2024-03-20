# Welcome to the community Agon Platform documentation

This is the community-driven documentation for the Agon Platform, covering the Agon Light, Olimex Agon Light 2, and the Agon Console8 hobbyist computers.  It is intended to be a comprehensive guide to the Agon Platform, including Agon MOS and VDP firmware, and the hardware and software that it supports.  It is intended to provide a guide to all of the features of the Agon platform with guidance on how to program for it, with information on the various features provided by different versions of the firmware.

This site is intended to supercede the official documentation, and to provide a more comprehensive and up-to-date guide to the Agon Platform.  It is intended to be a living document, and to be updated as new features are added to the platform.

The original documentation for Quark firmware can be found [here](https://github.com/breakintoprogram/agon-docs/wiki). This community documentation was started because GitHub's wiki system does not allow easy contributions by users, and the original documentation contains some errors and omissions.  We don't use the wiki here, but just markdown files which are also automatically generated into a [website](https://agonconsole8.github.io/agon-docs/).

## Want to contribute?

Great, please go to the [github repository](https://github.com/AgonConsole8/agon-docs) and create a Pull Request with your changes.

## What is the Agon Light, and what is the Agon Console8

The Agon Light is a modern, fully open-source, 8-bit microcomputer and microcontroller in one small, low-cost board, designed by Bernado Kastrup aka The Byte Attic. As a computer, it is a standalone device that requires no host PC: it puts out its own video (VGA), audio (2 identical mono channels), accepts a PS/2 keyboard and has its own mass-storage in the form of a µSD card.  The Olimex Agon Light 2 and the Agon Light Origins edition are variations on the original Agon Light design.

The Agon Console8 is a version of the Agon Light that also includes two Atari-compatible joystick ports, and a PS/2 mouse port, and a stylish case.  It was also designed by Bernado Kastrup aka The Byte Attic, and is manufactured by Heber Ltd.

The Agon Light, Olimex Agon Light 2, and the Agon Console8 are all fully compatible with each other.  Software written for one will run on the other, and the same firmware can be used on both.  In this documentation, generally, the term "Agon Light" is used to refer to all variations.

The main CPU is an eZ80F92, a modern Zilog Z80 microcontroller that is fully backwards compatible with the Z80. As well as running in a traditional 8-bit mode with a 64K address space, it can run in 24-bit mode with a 16MB address space, and is also capable of running in a hybrid mode with a mixture of 24-bit and 8-bit code.

The eZ80F92 integrates a number of standard peripherals, including a UART, and hardware timers.

There is a second CPU dedicated to handling video, sound, and keyboard, an ESP32-Pico-D4. This co-processor is linked to the eZ80F92 via a UART (a high speed serial communications link), and acts as a graphics terminal.

## What is the Quark Firmware

The Quark firmware is the official operating system for the Agon Light. It consists of three main components:

* [MOS](MOS.md): Machine Operating System
* [VDP](VDP.md): Visual Display Processor
* [BBC BASIC for Agon](BBC-BASIC-for-Agon.md): A specially adapted port of R.T.Russell's excellent BASIC interpreter

## How can I find out more?

The following documents provide some more information

* [Agon Platform theory of operation](Theory-of-operation.md)
* [External documentation](External-Documentation.md)
* [Using GPIO](GPIO.md)
* [The Agon Projects github](Projects.md)
* [Third party projects](Third-Party-Projects.md)
* [FAQ](FAQ.md)

## Where can I buy one?

- [Mouser](https://www.mouser.com/ProductDetail/Olimex-Ltd/AgonLight2?qs=9vOqFld9vZWAIti5ng59Vw%3D%3D)
- [The Pi Hut](https://thepihut.com/products/agonlight2-z80-bbc-basic-retro-single-board-computer)
- [Olimex Ltd](https://www.olimex.com/Products/Retro-Computers/AgonLight2/open-source-hardware)
- [DigiKey](https://www.digikey.com/en/products/detail/olimex-ltd/AGONLIGHT2/19204345)
- [PCBWay](https://www.pcbway.com/project/gifts_detail/Agon_light_3f7ffaa8.html)
- [Agon Light Australia](https://agonlight.au)
- [Agon Light Store (UK)](http://agon-light.store)
- [Agon Console8 from Heber (UK)](https://shop.heber.co.uk/agon-console8/)
