# What is the MOS

The MOS is a command line Machine Operating System, similar to CP/M or DOS, that provides a human interface to the Agon file system.

It also provides the [MOS API](MOS-API.md) for programmers to use that provides some basic facilities for file I/O and other common operations for BBC BASIC and other third-party applications.

This documentation explains the general features of MOS, as well as commands it offers and how to use them.  It covers the Quark 1.04 version of MOS, and the later Console8 MOS releases up to and including MOS 3.0.  Versions of MOS prior to Quark 1.04 may be missing some features described below.

## System Requirements

To get the most out of MOS, you will need the following:

* A 32GB or less micro-SD card formatted FAT32

Technically MOS will work without an SD card, but you won't be able to do much with it.

## Using an SD card

MOS supports the use of an SD card to store files and programs.  The SD card must be formatted as FAT32, and must be 32GB or less in size.  MOS supports automatically running a script file on boot and a way of adding new commands.

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

### Automatically running a script on boot

MOS has the ability to automatically run a script file on boot.  Quite what this script does is up to you, but it could be used to set up your Agon in a particular way, or to run a program automatically.

The exact file that will be run, and the manner in which the file is run varies depending on which version of MOS you are using.

Quark 1.04, and Console8 MOS 2.0.0 to 2.3.x will all at boot-up time look for a file named `autoexec.txt` in the root folder of the SD card.  If it finds one, it will read the file in, and execute the MOS commands in the file sequentially from top to bottom.

For example, to set keyboard to US, load BBC BASIC from the root folder, change to the test folder, then run BASIC

```
SET KEYBOARD 1
LOAD bbcbasic.bin
CD test
RUN
```

The manner in which this file is executed differs slightly between Quark 1.04 and the Console8 releases.  Quark will blindly execute the commands and silently carry on if there is an error, whereas Console8 releases will stop execution if there is an error in the file, report the error as well as which line the error occurred on.  MOS 2.x uses the [`Exec` command](mos/Star-Commands.md#exec) to run the file, whereas Quark uses a different method.

As of Console8 MOS 2.2.0 the autoexec.txt file will run on every boot.  Previous versions of MOS (including Quark 1.04) would only run the autoexec.txt file on a "hard reset" (i.e. a power cycle, or press of the reset button) and not on a soft reset (i.e. a `CTRL+ALT+DEL`).

As of Console8 MOS 3.0, at boot time the system will look for a file named `!boot.obey` in the root folder of the SD card, and run that file using the [`Obey` command](mos/Star-Commands.md#obey).  If that file is not found, it will look for a file named `autoexec.obey`.  If that is also not found it will fall back to looking for `autoexec.txt` and running that using the `Exec` command.  Only one boot file will be run, and the order of preference is `!boot.obey`, `autoexec.obey`, `autoexec.txt`.

## Soft Boot

Pressing `CTRL+ALT+DEL` will reboot MOS on the eZ80. (`CTRL+SHIFT+ESC` for MOS 1.02 or earlier)

NB:  This assumes that MOS is still talking to the VDP, as it is the VDP that is responsible for detecting keypresses.  Sometimes a soft reboot key combination will therefore not work, and you may need to instead press the reset button on your Agon.

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

MOS 3.0 allows you to customise the prompt using the `CLI$Prompt` [system variable](mos/System-Variables.md#cli).  By default this is set to a macro to display a prompt identical to that introduced in MOS 2.2.0.  If this variable is unset, then the prompt will revert to the simple `*` character.

## MOS Commands

MOS offsets a number of inbuilt commands that allow you to interact with the Agon file system and control your computer.

The various commands available in MOS are described in the [MOS Command Reference](mos/Star-Commands.md).

The command interpreter in MOS, as of MOS 3.0, has become fairly sophisticated.  It will support user-defined [commands aliases](mos/System-Variables.md#command-aliases), and the use of [system variables](mos/System-Variables.md).  As well as the ability to run [moslets](#moslets) and program files directly (as [described above](#the-run-path)) it can now also directly run any file that has a ["run type"](mos/System-Variables.md#file-type-variables) set up for it by simply typing the name of the file.

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

