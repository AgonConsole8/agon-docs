# Understanding the Agon Light computer platform

The Agon Light, and its variants, is a retro-style computer platform that is designed to be inexpensive and easy to manufacture, using commonly available parts.

The platform is designed to be easy to understand, and easy to program.  It is designed to be a platform that is suitable for learning about computers, and learning about programming.

Please note this is a preliminary guide to the Agon Light platform, and is subject to change.  The Agon Light platform is still under development, and this guide is still being written.


## The Agon Light hardware

### The main processor

The primary processor on which you run programs is an eZ80.  This is a microcontroller based on the Zilog Z80 processor, which was used in many computers and arcade machines in the 1970s and 1980s.  The eZ80 is a modern version of the Z80, with a few enhancements.  It has attached to it 512KB of RAM, which is considerably more than the 64KB that was common in the 1980s.  Also attached to the eZ80 chip is an SD card interface, allowing an SD card to be used for storage.

The eZ80 was chosen as the main processor of the Agon Light because of its historic ubiquity, and because it is still manufactured today.  It is a relatively simple processor, and is easy to understand and program.

### The video display processor, or VDP

Whilst the eZ80 is manufactured today, none of the video chips that were used in computers of the 1970s and 1980s are still manufactured, so for video output a different solution is needed.  To solve this problem the Agon Light uses a separate video processor, which we call the VDP, the Video Display Processor.  This chip is responsible for sound and video output, and also keyboard and mouse input.  Technically, this is an ESP32-Pico-D4 chip, which has attached to it an 8MB RAM chip which is used for screen memory and storing bitmaps and sound samples.

The exact technical details of the VDP however for most programmers is not important - in the Agon platform, from a software perspective, the VDP is designed to be used via a simple API, with the details of how it works abstracted away.  Indeed, when using an emulator of the Agon Light, the VDP chip is not emulated at all, and the emulator instead runs a compiled version of the VDP firmware using the host computer's video and audio hardware.


Effectively, the design of the Agon Light is that of two computers in one.  The eZ80 is the computer that is used to run programs, and the VDP is a computer that draws the screen and plays sound.  The eZ80 and the VDP communicate with each other using a high speed serial interface.


Some other retro-inspired computers take a different approach to solving the video output problem.  Often this involves making use of an FPGA chip, which is a chip that can be programmed to behave like any other chip.  This is a very flexible solution, but it is also very expensive.  The Agon Light takes a different approach, using a separate video processor chip, which is much cheaper than an FPGA.


The use of what is effectively a separate computer for the VDP in the Agon platform creates a significant architectural difference to many classic home computers of the 8-bit, 16-bit and early 32-bit era.  Those classic computers would usually use a shared area of memory that the main processor and the video processor would both have access to.  This shared memory would be used to store the screen memory, and the main processor would write to this memory to update the screen.  This is not the case with the Agon platform.  The eZ80 and the VDP have no shared memory, and the VDP has no access to the eZ80's RAM.  This means that the VDP cannot read the screen memory directly, and the eZ80 cannot write directly to the screen memory.  Instead, the VDP has to be sent commands to draw things to the screen, and the eZ80 has to be sent commands to read things from the screen.  This is a significant difference to many classic computers, and is something that programmers need to be aware of when writing programs for the Agon platform.


## The Agon software platform

From a software perspective, a significant inspiration for the Agon Light was the BBC Micro, a popular computer in the UK from the 1980s designed and built by Acorn Computers Ltd..  (Acorn went on to design the ARM processor, which is now the most commonly used processor design in the world.)  The BBC Micro was part of the BBC's Computer Literacy Project, which was a project to introduce computers to the UK population with an emphasis on education.  The BBC Micro was designed to be easy to use, and to be used in schools by children.

There are several significant differences between the Agon Light and the BBC Micro, but the two most important ones are the main processor, and the video system.  A BBC Micro was based on the 6502 CPU, which was a competitor to the Z80.  The 6502 was a simpler processor than the Z80, and was cheaper to manufacture, and given the relative simplicity often significantly faster than the Z80.  The 6502 was used in many popular computers of the 1970s and 1980s, including the Apple II, the Commodore 64, and the Atari 2600.  The Z80 was used in many other popular computers of the 1970s and 1980s, including the Sinclair ZX80, the Sinclair ZX81, the Sinclair Spectrum, the Amstrad CPC, and the MSX.  The BBC Micro however was designed from the ground up to be a computer capable of having multiple processors, and the first second processor option available for the computer was the Z80.  A version of BBC BASIC was produced for Acorn's Z80 second processor, and this is the version of BBC BASIC that the Agon Light is directly based on.


### BBC BASIC

An important part of the BBC Micro was BBC BASIC, a well respected version of the BASIC programming language.  Compared to other versions of BASIC, BBC BASIC was very powerful since it included many advanced features, such as procedures, functions, and recursion.  It also included many commands for graphics and sound, which made it easy to write games and other programs that used graphics and sound.

The original version of BBC BASIC for the 6502 processor was written by Sophie Wilson of Acorn.  The version of BBC BASIC for the Z80 was written by R.T.Russell, and was based on the original version by Sophie Wilson.  The version of BBC BASIC for the Agon Light is directly based on R.T.Russell's version, and is largely compatible with the BBC Micro version.

BBC BASIC has an inbuilt assembler, which allows for machine code to be embedded within BASIC programs.  This is a very powerful feature, and allows for programs to be written that make use of the full power of the processor.  Machine code can run very many times faster than interpreted BASIC.  It should be noted that the Z80 version of BASIC as used on the Agon platform has an assembler for the Z80 built in, whereas the original 6502 version of BASIC for the BBC Micro included an assembler for the 6502 processor.  Programs written for the BBC Micro that include assembler code will therefore not run on the Agon Light without rewriting those assembler routines.

#### BBC BASIC ADL version

There is now a newer version of BBC BASIC for the Agon platform known as the "ADL version".  The original Z80 version of BBC BASIC was restricted to work within 64KB of RAM.  This limitation was due to the fact that the Z80 processor only has 16 address lines (also referred to as a 16-bit address bus), so technically it can only address 64KB of RAM.

The newer eZ80 CPU used in the Agon Light has a 24-bit address bus, which means that it can potentially address up to 16MB of RAM.  Agon Light machines come with 512KB of RAM attached to the eZ80 CPU.

To allow programs to make use of this extra RAM, and some other new facilities, the eZ80 has extensions known as "ADL mode".  This mode extends some processor registers to 24 bits, and allows for the use of the extra RAM.  The ADL version of BBC BASIC is designed to make use of this extra RAM, and to allow for programs to be written that make use of more than 64KB of RAM.

The ADL version of BBC BASIC also includes modifications to the assembler to allow for the use of newer ADL machine code instructions.  This means that programs written for the ADL version of BBC BASIC may not run on the original Z80 version of BBC BASIC.


### VDU commands, and the VDP

The other significant difference between the Agon Light and the BBC Micro is the video system.  As video output is handled by a separate processor, and there is no shared memory between the two processors within an Agon Light, the design of the Agon system does not allow for the video system to work in the same kind of manner as a BBC Micro.  From a software perspective, however, the Agon Light is designed to be as similar as possible to the BBC Micro, and the video system is designed to be as similar as possible to the BBC Micro video system.  This means that many programs written for the BBC Micro can be ported to the Agon Light with minimal changes.

Using BASIC on a BBC Micro, the primary way to draw things to the screen was via simple keywords such as `PRINT`, `MOVE`, `DRAW`, `PLOT`, `COLOUR` (etc.) and most importantly `VDU`.  It is the `VDU` keyword that sent commands to the video system.  Effectively _all_ of the other keywords that drew onto the screen (or changed drawing settings) are just shorthand for `VDU` commands.

The same is true on the Agon Light.  The Agon's VDP processor has been written to understand the same `VDU` commands as the BBC Micro, and so the same keywords that draw things to the screen on a BBC Micro will also draw things to the screen on an Agon Light.  This means that many programs written in BBC BASIC for the BBC Micro can be ported to the Agon Light with minimal changes.

The Agon Light also includes a number of additional `VDU` commands that are not present on the BBC Micro, which allow for more advanced graphics and sound effects.  These commands are documented in the [VDP](VDP.md) section of this documentation.


The nature of the MOS/VDP split on the Agon platform carries with it some limitations that you should be aware of.

The nature of the VDP is that it is, essentially, a stream processor.  It accepts a stream of data from the eZ80, and interprets that as a stream of commands and data.  In regular use, there is no command to indicate the beginning of a command stream, and no command to indicate the end of a command stream.  The VDU commands that the VDP interprets are of a variable length, ranging from a single byte to potentially over 64kb.  A single `VDU` statement in BASIC could be a complete command from the perspective of the VDP, or it could be the first part of a command that is continued in the next `VDU` statement.  For commands that are intended to send across a large amount of data, such as a bitmap or a sound sample, the data is typically sent using a series of `VDU` statements, each of which is a part of the same command.

Care must therefore be taken to ensure that the VDP is sent complete commands, and that commands are not interleaved.  For instance, it may be tempting when transfering over a sound sample to use `PRINT` or `DRAW` statements to show progress.  This will not work, because the VDP will interpret the `PRINT` or `DRAW` statements (which will have been translated into a raw VDU byte command stream) as part of the sound sample data.

To solve this issue, the VDP provides a [Buffered Commands API](VDP---Buffered-Commands-API.md).  This allows for data to be sent to the VDP in discrete chunks, sent to one of over 65000 buffers.  Once all of the data has been sent, a command can be sent to the VDP to make use of the data in the buffer, whether that is to use the buffer as a sound sample, a bitmap, or a sequence of VDU commands.  So to print a progress bar whilst sending a sound sample, you would send the sound sample data to a buffer a chunk at a time using the Buffered Commands API "add block" command, and after each chunk use drawing commands to update the progress bar.  Once all of the sound sample data has been sent, you would then send a command to the VDP to identify that data as a sound sample, and/or use other commands to play that sample.






