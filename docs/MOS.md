# What is the MOS

The MOS is a command line Machine Operating System, similar to CP/M or DOS, that provides a human interface to the Agon file system.

It also provides the [MOS API](MOS-API.md) for programmers to use that provides some basic facilities for file I/O and other common operations for BBC BASIC and other third-party applications.

This documentation explains the general features of MOS, as well as commands it offers and how to use them.  It covers the Quark 1.04 version of MOS, and the later Console8 MOS releases up to and including MOS 3.0.  Versions of MOS prior to Quark 1.04 may be missing some features described below.

Please note that if you are running Quark 1.04 or earlier, the capabilities of MOS are very limited, and quite a few features described in this documentation will not be available to you.  You are strongly advised to upgrade to a later version of MOS.  MOS 2.x and MOS 3.x releases are fully compatible with software written for Quark 1.04, and will run all Quark 1.04 software without modification.

## System Requirements

To get the most out of MOS, you will need the following:

* A micro-SD card formatted as FAT32

Technically MOS will work without an SD card, but you won't be able to do much with it.

## Using an SD card

MOS supports the use of an SD card to store files and programs.  The SD card must be formatted as FAT32.  This documentation previously advised that the card should be 32GB or less in size, however several users report successfully using 64GB cards.  MOS supports automatically running a script file on boot and a way of adding new commands.

### Moslets

"Moslets" are a special type of program that are intended to extend the functionality of MOS by adding in new commands.

By convention, moslets are stored in a special folder named `mos` at the top level of the SD card.  When you attempt to use a command that is not built into MOS, the system will look in the `mos` folder for a file with the same name as the command with a `.bin` extension.  If it finds one, it will execute that file as if it were a built-in command.

Moslets have a special requirement in that they must be built to run from memory address `0x0B0000` onwards.  The use of this address is intended to allow moslets to run without affecting the main user program space.  The idea being that this will let you use a moslets from within BASIC (or from within other programs) without having to worry about it overwriting your program.

From MOS 3.0 onwards it is possible to specify a different directory, or multiple directories, for the location of moslets on your system.  This is done by setting the `Moslet$Path` [system variable](mos/System-Variables.md#system-path-variables).  By default this variable is set to `/mos/` for compatibility with previous versions of MOS.

### The "run path"

Console8 MOS 2.2.0 effectively added the concept of a "run path" to MOS, which would be used by the CLI to either search for commands not built into MOS, or to run programs.  In MOS 2.2.0 to MOS 2.3.2 the order of directories searched would be fixed, with the `mos` folder being searched first for moslets, followed by the current directory, and then the `bin` folder.  In MOS 2.x the order could not be changed.  MOS 3.0 provides a way to change this.

Programs located outside of the moslet folder are expected to be full standalone programs that will be run and executed at the default memory address of `0x040000`, and thus will overwrite existing programs.

NB there is currently no way to control the memory address that a program is automatically loaded into, so if you have a program that needs to be loaded at a specific address, you will need to use the `Load` command to load it manually and then use the `Run` command to start the program.

It should be noted that MOS will always run the first program it finds with the same name as the command you have entered.  This means for example that if you have a moslet with the same name as a program in the `bin` folder, the moslet will be run instead of the program.

From MOS 3.0 onwards the order of directories searched is controlled by the `Run$Path` [system variable](mos/System-Variables.md#system-path-variables).  By default this is a macro variable set to `<Moslet$Path>, ./, /bin/` to be compatible with previous versions of MOS.  This variable can be changed to search in any order you like, and can include multiple directories.  By changing this variable you can, for example, make it so that the `bin` folder is searched before the `mos` folder, or add additional directories to search in.

### Automatically running a script on boot {#boot-script}

MOS has the ability to automatically run a script file on boot (power on, or a system reset).  Quite what this script does is up to you, but it could be used to set up your Agon in a particular way, or to run a program automatically.

The exact file that will be run, and the manner in which the file is run varies depending on which version of MOS you are using.

As of MOS 3, the system will first look for a boot file named `!boot.obey`, and use that as it's preferred boot script, running it using the [`obey` command](mos/Star-Commands.md#obey).  If that file is not present, it will look for a file named `autoexec.obey` and run that.  If neither of these are present it will fall back to working in a similar manner to Quark MOS 1.04 and MOS 2.x.  If you are running MOS 3, and have a suitably up-to-date version of the VDP firmware installed, you can prevent the boot script from being run by holding down the left "Shift" key on your keyboard during power up or reset.

Quark 1.04, and Console8 MOS 2.x releases will all, at boot-up time, look for a file named `autoexec.txt` in the root folder of the SD card.  The manner in which this file is executed differs slightly between Quark 1.04 and the later Console8 releases.  Quark will blindly execute the commands and silently carry on it encounters an error, and any such errors will not be reported.  The Console8 MOS 2.x releases execute the `autoexec.txt` file using the [`Exec` command](mos/Star-Commands.md#exec) and will stop execution if there is an error in the file, reporting the error as well as which line the error occurred on.

If no matching script file is found, the system will simply boot to the command prompt.

An example script, compatible with Quark 1.04 and all later versions of MOS, follows.  This script will to set keyboard to US, load BBC BASIC from the root folder, change to the test folder, then run BASIC.

```
SET KEYBOARD 1
LOAD bbcbasic.bin
CD test
RUN
```

If you are running MOS 2.2.0 or later, and `bbcbasic.bin` is in your run path (most likely in a `/bin` directory), this script could be simplified to:

```
SET KEYBOARD 1
cd test
bbcbasic
```

MOS 3.0 allows for a great deal of customisation of your MOS environment, including the ability to set up custom command aliases, set "run types" to allow all sorts of files to be directly run from the command line, a custom command prompt, adjust the "run path" that will be searched for commands and executables, and many other features.  Many of these features are controlled through the use of [system variables](mos/System-Variables.md).  It is common to have a `!boot.obey` that will set up system variables to customise your MOS environment.

MOS 3.0's preferred use of `!boot.obey` means that if you have more than one Agon computer running different versions of MOS then you can use the same SD card in all of them.  This would allow your MOS 3 system to use commands in its `!boot.obey` file that are not available in Quark MOS 1.04 or MOS 2, and those systems will still run the separate `autoexec.txt` file.  Your `!boot.obey` file could include commands specific to MOS 3, and finish with the command `exec autoexec.txt` to run the boot file used by earlier versions of MOS.

## Soft Boot

Pressing `CTRL+ALT+DEL` will reboot MOS on the eZ80. (`CTRL+SHIFT+ESC` for MOS 1.02 or earlier)

NB:  This assumes that MOS is still talking to the VDP, as it is the VDP that is responsible for detecting keypresses.  Sometimes a soft reboot key combination will therefore not work, and you may need to instead press the reset button on your Agon.

If you are using a MOS version earlier than 2.2.0, then a soft reset will not try to run the boot script.  To make sure the boot script runs you would need to do a "hard boot" by powering cycling your Agon (turn it off, then on again), or pressing the reset button.

## The MOS Command Line Interface

MOS provides a command line interface (CLI) that allows you to interact with the Agon file system and perform some basic control over your Agon computer.

The MOS CLI is loosely inspired by the Acorn MOS present in the BBC Micro and later Acorn computer systems like the Archimedes.  The features and facilities supported by the MOS command line have evolved over time, with MOS 3.0 supporting many more commands than earlier versions of MOS.

MOS works alongside the Agon's VDP, using the facilities of the VDP to display text on the screen and accept input from the keyboard.  The VDP provides some useful facilities, such as a "paged mode" that will stop the screen from scrolling until you press the `SHIFT` key to continue, or `ESCAPE` to exit.  Paged mode can be toggled on and off by pressing `CTRL+N` and `CTRL+O` respectively.

### The MOS line editor

MOS provides a simple line editor that allows you to edit the current command line before submitting it to the system.  This same line editor is available for third-party applications like BBC BASIC to use.

The line editor allows you to move the cursor around the current line of text, insert and delete characters, and submit the line to the system.  Whilst much of the functionality on Agon is inspired by the BBC Micro, the line editor differs - there is no "copy" cursor system on the Agon.  This line editor is similar to those that you will find on modern operating systems like MacOS, Linux or Windows.

The MOS CLI line editor will also provide some basic command history, keeping track of the last 16 commands the user has entered.  Pressing the `UP` arrow key when at the beginning of a line will replace the current line with the last entered command.  Similarly pressing the `DOWN` arrow key at the end of a line will cycle through the command history in the opposite direction.  The `HOME` and `END` keys will move the cursor to the start and end of the current line respectively.

The Console8 MOS 2.2.0 release also adds support for pressing the `PAGE UP` and `PAGE DOWN` keys to quickly step through the command history.

Also added to the 2.2.0 release is "tab completion".  If you start typing a command and then press the `TAB` key, MOS will attempt to complete the command for you.  This includes both built-in commands, moslets found in the `mos` folder, programs found in the current directory, and programs found in the `bin` folder, or file names within the current directory.  In MOS 3.0 onwards, the tab completion uses the `Run$Path` variable to determine where to look for commands.

There is also support for programmable function keys in the 2.2.0 release.  For more information on that see the [`Hotkey` command](mos/Star-Commands.md#hotkey).

### The MOS command prompt

On versions of MOS prior to the 2.2.0 release, the MOS command prompt is a simple `*` character.  This is the point at which you can enter commands to the system.

From the Console8 MOS 2.2.0 release onwards the prompt has been extended to include the current directory.  This is to help you keep track of where you are in the file system.  The prompt will look something like this:

```
/programs *
```

MOS 3.0 allows you to customise the prompt using the [`CLI$Prompt` system variable](mos/System-Variables.md#cli).  By default this is set to a macro to display a prompt identical to that introduced in MOS 2.2.0.  If this variable is unset, then the prompt will revert to the simple `*` character.

## MOS Commands

MOS offsets a number of inbuilt commands that allow you to interact with the Agon file system and control your computer.

The various commands available in MOS are described in the [MOS Command Reference](mos/Star-Commands.md).  The exact array of built-in commands available has changed over time with new commands being added, and new features added to existing commands.

As of MOS 3.0 the command interpreter has become fairly sophisticated.  It will support user-defined [commands aliases](mos/System-Variables.md#command-aliases), and the use of [system variables](mos/System-Variables.md).  As well as the ability to run [moslets](#moslets) and program files directly (as [described above](#the-run-path)) it can now also directly run any file that has a ["run type"](mos/System-Variables.md#file-type-variables) set up for it by simply typing the name of the file.

## MOS System Variables

From MOS 3.0 onwards, MOS supports the concept of [system variables](mos/System-Variables.md).  These can be used for user programs, as well as providing ways to gain information about the system and control the behaviour of MOS.

## Script files

All Console8 versions of MOS support the concept of a script file.  This is a file that contains a series of MOS commands that can be executed in sequence.  This is useful for automating tasks, or for setting up your Agon in a particular way.  When a script file is run, if an error is encountered the system will stop executing the script and report the error, as well as which line the error occurred on.

The first type of script file is an "Exec" file, which is a simple text file containing a series of MOS commands, one per line.  You can run an Exec file using the [`Exec` command](mos/Star-Commands.md#exec).

MOS 3.0 has added a second type of script file, which  is an "Obey" file, which are run using the [`Obey` command](mos/Star-Commands.md#obey).  This is very similar to an Exec file, but when running an Obey file the system will set a [system variable](mos/System-Variables.md#obey-files) to indicate the directory inside which the Obey file is located, and it will also support [argument substitution](mos/Argument-Substitution.md).

Quark MOS 1.04 only supports a single script file, `autoexec.txt` that will be run at boot, and offers no command to run a script file.


## Memory map

Addresses are 24-bit, unless otherwise specified

- `&000000 - &01FFFF`: MOS (Flash ROM)
- `&040000 - &0BDFFF`: User RAM
- `&0B0000 - &0B7FFF`: Storage for loading MOS star command executables off SD card
- `&0BC000 - 0BFFFFF`: Global heap and stack


## The Stack

Up to and including the MOS 3.0 release, MOS sets up a stack pointer for its own use.  As MOS is written to run in the eZ80's "ADL" mode, this means the `SPL` register, the version of `SP` visible to ADL-mode code, is what is set up.  Programs that are run in ADL-mode will see this stack pointer when they start up.  Care should therefore be taken to ensure that the stack pointer is not corrupted by the program, as this will affect the operation of MOS and any other programs that are run after it.

The architecture of the eZ80 means that programs running in Z80 mode see a different version of the stack pointer `SP` register, known as `SPS`.  This value is _not_ set up by MOS, and is left to the user program to set up.  The value of `SP` in Z80 mode should be treated by programs as undefined on startup.

Future versions of MOS may change the behaviour of the stack pointer for ADL-mode code in order to provide better protection for the MOS stack.  For compatibility with existing software, a stack pointer will still be set up for programs that use ADL-mode, but this may be allocated within the MOS heap space.
