# Welcome to the community Agon Console8 documentation

This is not the offical documentation for Agon Light or its firmware Quark. You can find it [here](https://github.com/breakintoprogram/agon-docs/wiki). This community documentation was started, because GitHub wiki does not allow easy contributions by users. We don't use the wiki here, but just markdown files which are also automatically generated into a [website](https://agonconsole8.github.io/agon-docs/).

## Want to contribute?

Great, please go to the [github repository](https://github.com/AgonConsole8/agon-docs) and create a Pullrequest with your changes.

## What is the Agon Light

The Agon Light is a modern, fully open-source, 8-bit microcomputer and microcontroller in one small, low-cost board, designed by Bernado Kastrup aka The Byte Attic. As a computer, it is a standalone device that requires no host PC: it puts out its own video (VGA), audio (2 identical mono channels), accepts a PS/2 keyboard and has its own mass-storage in the form of a µSD card.

The main CPU is an eZ80F92, a modern Zilog Z80 microcontroller that is fully backwards compatible with the Z80. As well as running in a traditional 8-bit mode with a 64K address space, it can run in 24-bit mode with a 16MB address space, and is also capable of running in a hybrid mode with a mixture of 24-bit and 8-bit code.

The eZ80F92 integrates a number of standard peripherals, including a UART, and hardware timers.

There is a second CPU dedicated to handling video, sound, and keyboard, an ESP32. This co-processor is linked to the eZ80F92 via a UART, and acts as a graphics terminal.

## What is the Quark Firmware

The Quark firmware is the official operating system for the Agon Light. It consists of three main components:

* [MOS](MOS.md): Machine Operating System
* [VDP](VDP.md): Visual Display Processor
* [BBC BASIC for Agon](BBC-BASIC-for-Agon.md): A specially adapted port of R.T.Russell's excellent BASIC interpreter

## Where can I buy one?

- [Mouser](https://www.mouser.com/ProductDetail/Olimex-Ltd/AgonLight2?qs=9vOqFld9vZWAIti5ng59Vw%3D%3D)
- [The Pi Hut](https://thepihut.com/products/agonlight2-z80-bbc-basic-retro-single-board-computer)
- [Olimex Ltd](https://www.olimex.com/Products/Retro-Computers/AgonLight2/open-source-hardware)
- [DigiKey](https://www.digikey.com/en/products/detail/olimex-ltd/AGONLIGHT2/19204345)
- [PCBWay](https://www.pcbway.com/project/gifts_detail/Agon_light_3f7ffaa8.html)
- [Agon Light Australia](https://agonlight.au)
- [Agon Light Store (UK)](http://agon-light.store)
- [Agon Console8 from Heber (UK)](https://shop.heber.co.uk/agon-console8/)

Please note that the Olimex version (Agon Light 2) is lightly customised, yet still fully compatible with the original Agon Light.
