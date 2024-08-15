# What is the MOS

The MOS is a command line Machine Operating System, similar to CP/M or DOS, that provides a human interface to the Agon file system.

It also provides the [MOS API](MOS-API.md) for programmers to use that provides some basic facilities for file I/O and other common operations for BBC BASIC and other third-party applications.

This documentation explains the general features of MOS, as well as commands it offers and how to use them.  It covers the Quark 1.04 version of MOS, and the later Console8 MOS versions up to and including 2.2.3.  Versions of MOS prior to Quark 1.04 may be missing some features described below.

## System Requirements

To get the most out of MOS, you will need the following:

* A 32GB or less micro-SD card formatted FAT32

Technically MOS will work without an SD card, but you won't be able to do much with it.

## Using an SD card

On powering on, your Agon will automatically mount the SD card and look for an `autoexec.txt` file in the root folder. If it finds one, it will execute the commands in the file.

MOS also supports a `mos` folder to add in some additional commands, and from Console8 MOS 2.2.0 onwards it also supports a `bin` folder for executables.

### The `mos` folder

The `mos` folder is a special folder that must be placed at the top level of your SD card that can be used to extend the functionality of MOS.

When you attempt to use a command that is not built into MOS, it will look in the `mos` folder on the SD card for a file with the same name as the command with a `.bin` extension. If it finds one, it will execute that file as if it were a built-in command.

Commands provided via the `mos` folder are known as "moslets".  These have a special requirement in that they must be built to run from memory address `0x0B0000` onwards.  The use of this address is intended to allow moslets to run without affecting the main user program space.  The idea being that this will let you use a moslets from within BASIC without having to worry about it overwriting your program.

### The `bin` folder

From Console8 MOS 2.2.0 onwards, MOS will also support a `bin` folder at the top level of your SD card.

The `bin` folder can be used to store programs that can be run from MOS.  These differ from moslets in that they are expected to be full standalone programs that will be run and executed at the default memory address of `0x040000`, and thus will overwrite existing programs.

If you have attempted to use a command that was not built into MOS, and was not found in the `mos` folder, MOS will then look in the `bin` folder for a file with the same name as the command with a `.bin` extension. If it finds one, it will load and run that file.

NB there is currently no way to control the memory address that a program in the `bin` folder will be loaded to, so if you have a program that needs to be loaded at a specific address, you will need to use the `LOAD` command to load it manually.  Additionally there is no way to control the manner in which MOS searches for commands, so if you have a program in the `bin` folder that you want to run, you will need to ensure that there is no built-in command or "moslet" with the same name.  These limitations may be changed in the future.

### The `autoexec.txt` file

If, at boot-up time, MOS detects an `autoexec.txt` file in the root folder of the SD card, it will read the file in, and execute the MOS commands in the file sequentially from top to bottom.

For example, to set keyboard to US, load BBC BASIC from the root folder, change to the test folder, then run BASIC

```
SET KEYBOARD 1
LOAD bbcbasic.bin
CD test
RUN
```

The manner in which this file is executed differs slightly between Quark 1.04 and the Console8 releases.  Quark will blindly execute the commands and silently carry on if there is an error, whereas Console8 will stop execution if there is an error in the file, report the error as well as which line the error occurred on.

As of Console8 MOS 2.2.0 the autoexec.txt file will run on every boot.  Previous versions of MOS (including Quark 1.04) would only run the autoexec.txt file on a "hard reset" (i.e. a power cycle, or press of the reset button) and not on a soft reset (i.e. a `CTRL+ALT+DEL`).

## Soft Boot

Pressing `CTRL+ALT+DEL` will reboot MOS on the eZ80. (`CTRL+SHIFT+ESC` for MOS 1.02 or earlier)

NB:  This assumes that MOS is still talking to the VDP, as it is the VDP that is responsible for detecting keypresses.  Sometimes a soft reboot key combination will therefore not work, and you may need to instead press the reset button on your Agon.

## The MOS Command Line Interface

MOS provides a command line interface (CLI) that allows you to interact with the Agon file system and perform some basic control over your Agon computer.

The MOS CLI is loosely inspired by the Acorn MOS present in the BBC Micro and later Acorn computer systems like the Archimedes.

MOS works alongside the Agon's VDP, using the facilities of the VDP to display text on the screen and accept input from the keyboard.  The VDP provides some useful facilities, such as a "paged mode" that will stop the screen from scrolling until you press the `SHIFT` key to continue, or `ESCAPE` to exit.  Paged mode can be toggled on and off by pressing `CTRL+N` and `CTRL+O` respectively.

### The MOS line editor

MOS provides a simple line editor that allows you to edit the current command line before submitting it to the system.  This same line editor is available for third-party applications like BBC BASIC to use.

The line editor allows you to move the cursor around the current line of text, insert and delete characters, and submit the line to the system.  Whilst much of the functionality on Agon is inspired by the BBC Micro, the line editor differs - there is no "copy" cursor system on the Agon.  This line editor is similar to those that you will find on modern operating systems like MacOS, Linux or Windows.

The MOS CLI line editor will also provide some basic command history, keeping track of the last 10 commands the user has entered.  Pressing the `UP` arrow key when at the beginning of a line will replace the current line with the last entered command.  Similarly pressing the `DOWN` arrow key at the end of a line will cycle through the command history in the opposite direction.  The `HOME` and `END` keys will move the cursor to the start and end of the current line respectively.

The Console8 MOS 2.2.0 release also adds support for pressing the `PAGE UP` and `PAGE DOWN` keys to quickly step through the command history.

Also added to the 2.2.0 release is "tab completion".  If you start typing a command and then press the `TAB` key, MOS will attempt to complete the command for you.  This includes both built-in commands, moslets found in the `mos` folder, programs found in the current directory, and programs found in the `bin` folder.

There is also support for programmable function keys in the 2.2.0 release.  For more information on that see the HOTKEY command.

### The MOS command prompt

On versions of MOS prior to the 2.2.0 release, the MOS command prompt is a simple `*` character.  This is the point at which you can enter commands to the system.

From the Console8 MOS 2.2.0 release onwards the prompt has been extended to include the current directory.  This is to help you keep track of where you are in the file system.  The prompt will look something like this:

```
/programs *
```

(It is intended that future versions of MOS will allow you to customise the prompt to your liking.)

## MOS Commands

MOS offers a number of commands that allow you to interact with the Agon file system and control your computer.  These commands are entered at the MOS command prompt, and are executed by pressing the `RETURN` key.  If you are inside BBC BASIC, you can also enter MOS commands by preceding them with an asterisk `*`.

MOS commands are case-insensitive, and can be abbreviated with a dot `.`.  For example, `DELETE myfile` and `DEL. myfile` are equivalent.  When you abbreviate a command, the first matching command is used, so whilst there are several commands that begin with `C`, `C.` will execute the `CAT`.  Commands that accept multiple parameters will expect those parameters to be space-delimited.  Numbers are in decimal, but can be prefixed with an ampersand `&` for hexadecimal.

To aid users coming from other systems, several commands have aliases, for example `DELETE` and `ERASE` are equivalent.  The aliases are listed in the command descriptions below.

Commands are described below.  Commands parameters are described with angle brackets `<param>` and optional parameters are indicated with square brackets `[<param>]` or `[-flag]`, and alternative parameters are shown either side of a `|` character.

Any command that requires a memory address will expect a 24-bit address value.

The commands available in MOS are as follows:

### `.` (dot)

Syntax: `*. [<path>]` (Alias for `CAT`)

This command is an alias for the `CAT` command.  If a path is provided, it will list the contents of that directory, otherwise it will list the contents of the current directory.

### `CAT`

Syntax: `*CAT [-l] [<path>]` (Aliases include `DIR` and `.`)

The "Catalogue" command, which displays a directory listing of the current directory, or of the directory at the path given.

From Console8 MOS 2.2.0 onwards, the `-l` flag can be used to display the long form of the directory listing, which includes the file size and date/time of last modification, otherwise a shorter form of the directory listing will be displayed.  Versions of MOS prior to 2.2.0 will always display the long form of the directory listing.

### `CD`

Syntax: `*CD <path>`

Change current directory.

The path provided to the `CD` command uses `/` as the directory separator, and can be either an absolute path or a relative path.  The root directory is `/`.  `CD ..` can be used to move up one directory level.

### `CDIR`

Syntax: `*CDIR <path>`

This command is an alias for the `CD` command.

Available from Console8 MOS 2.2.0 onwards.

### `CLS`

Syntax: `*CLS`

Clear the screen.

(This command performs a VDU 12, which is the same as the `CLS` command in BBC BASIC.)

### `COPY`

Syntax: `*COPY <source> <destination>`

Create a copy of a file.

From Console8 MOS 2.2.0 onwards, the `COPY` command will also support the use of wildcards in the source file name, such as `*.txt`, and can support the destination being a directory.

### `CP`

Syntax: `*CP <source> <destination>`

This command is an alias for the `COPY` command.

Available from Console8 MOS 2.2.0 onwards.

### `CREDITS`

Syntax: `*CREDITS`

Outputs some credits and version numbers for third-party libraries used in the Agon firmware.

### `DELETE`

Syntax: `*DELETE <filename>` (Aliases include `ERASE`)

Delete a file or folder (must be empty).

### `DIR`

Syntax: `*DIR [<path>]`

This command is an alias for the `CAT` command.

### `ERASE`

Syntax: `*ERASE <filename>`

This command is an alias for the `DELETE` command.

### `EXEC`

Syntax: `*EXEC <filename>`

Executes commands from a text file.

Commands will be executed one at a time.  If an error occurs processing a command, execution will stop and the error will be reported.

(This is essentially how the `autoexec.txt` file is processed at boot time.)

NB: This command is not available in Quark 1.04.

### `HELP`

Syntax: `*HELP [<command> | all]`

Displays help information for a command.  If no command is provided, a list of available commands will be displayed.

NB prior to Console8 MOS 2.2.0 the `HELP` command required a command name to be provided, or the `all` keyword to display all commands.

### `HOTKEY`

Syntax: `*HOTKEY [<n> [<command string>]]`

Sets or clears programmable function keys.

Using this command you can set the function keys `F1-F12` on your to perform a specific command.  There are three ways in which this command works:
1. If no parameters are provided, then the current hotkeys will be displayed.
2. If a function key number `n` is provided alone without a command string, then the current command assigned to that function key will be cleared.
3. If a function key number `n` and a command string are provided, then the command string will be assigned to that function key.
    - If the command string includes `%s` then the current input line will be substituted in place of `%s`.

### `JMP`

Syntax: `*JMP <address>`

Jumps to the specified address in memory.  Using this command, you can jump to any address in memory, including the ROM.  If used from the MOS command line, the processor will be in the eZ80 ADL mode.

### `LOAD`

Syntax: `*LOAD <filename> [<address>]`

Load a file from the SD card into the specified address in memory.  If no address parameters is specified, then the address will default to `&40000`.

### `MKDIR`

Syntax: `*MKDIR <name>`

Make a new directory on the SD card.

### `MOUNT`

Syntax: `*MOUNT`

Mount the SD card.

If you have ejected your SD card, you can use this command to re-mount it.

### `MOVE`

Syntax: `*MOVE <source> <destination>`

This command is an alias for the `RENAME` command.

### `MV`

Syntax: `*MV <source> <destination>`

This command is an alias for the `RENAME` command, and is only available from Console8 MOS 2.2.0 onwards.

### `RENAME`

Syntax: `*RENAME <source> <destination>` (Aliases include `MOVE`)

This is the command to rename a file or move a file to a different folder.

To rename a file in the same folder.

`*RENAME autoexec.txt autoexec.bak`

Rename a file and move to a different folder (the destination folder must exist).

`*RENAME test.bas archive/test.bas`

From Console8 MOS 2.2.0 onwards, the `RENAME` command also supports allowing the destination to specify a directory.  If the destination is a directory, the source file will be moved to that directory and keep the same filename.  Additionally if the destination is a directory then the source file specified can include a wildcard, such as `*.txt`.

### `RM`

Syntax: `*RM <filename>`

This command is an alias for the `DELETE` command, and is only available from Console8 MOS 2.2.0 onwards.

### `RUN`

Syntax: `*RUN <address | .> [<parameters>]`

Call an executable binary loaded in memory. If no parameters are passed, then the address will default to `&40000`.  The address parameter can also be replaced with `.` to indicate that the default address of `&40000` should be used.  Any additional parameters will be passed through to the executable.

When using the `RUN` command MOS will check the header of the executable to determine which mode it is in, and will run the executable in the appropriate mode.  This allows you to run both Z80 mode executables, which are restricted to 64kb of RAM, and ADL mode executables which can use all 512kb of memory.  The header starts at byte 64 of the executable, which must contain `MOS`.  Byte 68 must be a `0` byte to indicate Z80 mode, or a `1` byte to indicate ADL mode.

When a program is executed using the `RUN` command, MOS will set up the following processor registers:

- `A` will be set to the current memory bank value `MB`
- `DE(U)` the execution address passed to `RUN`
- `HL(U)` pointer to additional parameters passed to `RUN`

### `SAVE`

Syntax: `*SAVE <filename> <address> <size>`

Save a block of memory to the SD card.

### `SET`

Syntax: `*SET <option> <value>`

Set a system option.

NB before Console8 MOS 2.2.0 the `SET` command was case-sensitive for the option name.

#### Keyboard Layout

Syntax: `*SET KEYBOARD <n>`

Sets the current system keyboard layout.

- 0: UK (default)
- 1: US
- 2: German
- 3: Italian
- 4: Spanish
- 5: French
- 6: Belgian
- 7: Norwegian
- 8: Japanese
- 9: US International *
- 10: US International (alternative) *
- 11: Swiss (German) *
- 12: Swiss (French) *
- 13: Danish *
- 14: Swedish *
- 15: Portuguese *
- 16: Brazilian Portuguese *
- 17: Dvorak *

\* These layouts are only supported from Console8 MOS 2.2.0 onwards.  The exact keyboard layouts supported will depend on the version of the VDP firmware you have installed.  Prior to Console8 MOS 2.2.0 keyboard values of 9 or above would be ignored.  From Console8 MOS 2.2.0 onwards any keyboard value can be specified - if the keyboard layout is not supported by the VDP firmware then it will be ignored.

#### Console Mode

Syntax: `*SET CONSOLE <n>`

Sets the console mode, where if `n` is `0` then console mode will be disabled, and if `n` is `1` then console mode will be enabled.

When console mode is enabled, all (initial) "command" and text bytes sent to the VDP will be echoed out of the VDP to any attached USB serial device.  This can be useful for debugging purposes.

It is recommended that instead of using this command, if you are running Console8 VDP 2.5.0 or later, you should use `VDU 2` instead to enable the "printer", which also outputs from the VDP to the same serial device.  The main difference is that the "printer" device will filter out any control codes, providing a safer output, whereas the console mode will output everything.  The printer can be disabled with `VDU 3`.  Additionally pressing `CTRL+B` on the keyboard will enable the printer, `CTRL+C` will disable it, and `CTRL+P` will toggle it.

### `TIME`

Syntax:
- `*TIME`
- `*TIME <yyyy> <mm> <dd> <hh> <mm> <ss>`

Set and read the ESP32 real-time clock

### `TYPE`

Syntax: `*TYPE <filename>`

Display the contents of a text file on the screen.

### `VDU`

Syntax: `*VDU <char1> <char2> ... <charN>`

Write a stream of character bytes to the VDP.  This can be used to perform various control functions, such as clearing the screen, changing the screen mode, setting the cursor position, or changing the text colour.  More information on VDU commands can be found in the [VDP documentation](VDP.md).

This command is similar to the `VDU` command in BBC BASIC, but instead of separating arguments with commas you must instead separate them with spaces.

Prior to Console8 MOS 2.2.0, the `VDU` command would only accept 8-bit numbers as arguments.  Any values provided greater than 255 would be ignored.

From Console8 MOS 2.2.0 onwards this command will now also support 16-bit numbers as arguments, and several different ways to indicate hexadecimal numbers.  The BBC BASIC standard of suffixing a number with a semicolon character `;` to indicate a 16-bit number is supported.  As well as the `&` prefix for hexadecimal numbers, you can also use `0x` as a prefix, or `h` as a suffix.  Hexadecimal numbers provided greater than 255 will always be treated as 16-bit numbers.


## Memory map

Addresses are 24-bit, unless otherwise specified

- `&000000 - &01FFFF`: MOS (Flash ROM)
- `&040000 - &0BDFFF`: User RAM
- `&0B0000 - &0B7FFF`: Storage for loading MOS star command executables off SD card
- `&0BC000 - 0BFFFFF`: Global heap and stack

