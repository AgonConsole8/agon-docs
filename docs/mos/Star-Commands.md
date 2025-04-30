# MOS Commands

MOS contains a number of inbuilt commands that are always available to the MOS CLI.  These are sometimes referred to as "star commands" because the MOS command line interpreter has historically used a `*` character as its command prompt, and additionally these commands can be used from inside BBC BASIC by prefixing them with a `*` character.

MOS commands are case-insensitive, and can be abbreviated with a dot `.`.  For example, `DELETE myfile` and `DEL. myfile` are equivalent.  When you abbreviate a command, the first matching command is used, so whilst there are several commands that begin with `C`, `C.` will execute the `CAT`.  Commands that accept multiple parameters will expect those parameters to be space-delimited.

You should note that across versions of MOS, command abbreviations may change, as new commands may be added.  When writing script files, or using commands in BBC BASIC programs, it is recommended to use the full command name to avoid any potential issues with command abbreviations changing.

Commands that expect a number will, by default, expect that number to be given in decimal.  To specify a hexadecimal number, prefix it with an ampersand `&`.  For example, `*JMP &8000` will jump to address `&8000`.

MOS 3 extends the number format support of MOS 2, allowing numbers to be provided in various other formats.  Hexadecimal numbers can be specified with a `0x` prefix, or an `h` suffix.  Numbers in any numeric base from 2 to 36 can also be used by prefixing the number with the base (in decimal) followed by an underscore and then the number.  For instance, `2_111` indicates the binary number `111` (7 in decimal), `16_41` is hexadecimal number `41` (65 in decimal), and `8_72` is octal `72` (58 in decimal).

To aid users coming from other systems, several commands have aliases built in, for example `DELETE` and `ERASE` are equivalent.  The aliases are listed in the command descriptions below.

As of MOS 3.0, MOS also supports [user-defined aliases](System-Variables.md#command-aliases).  For compatibility the existing aliases supported in MOS 2 are maintained.  MOS 3.0 also allows the use of [system variables](System-Variables.md) in the command line.  For information on how variables are interpreted and expanded in commands, see the [`echo` command](#echo).  Support for [custom files paths](System-Variables.md#path-variables) in commands, which can be used for any command that requires a file or directory name, is also now included.

The various commands MOS supports are described below.  Commands parameters are described with angle brackets `<param>` and optional parameters are indicated with square brackets `[<param>]` or `[-flag]`, and alternative parameters are shown either side of a `|` character.

Any command that requires a memory address will expect a 24-bit address value.

The commands available in MOS are as follows:

## `%`

Syntax: `*%<command>`

Prefixing any command with the `%` character makes the command interpreter skip [command alias](System-Variables.md#command-aliases) resolution.  This means for example that you could redefine an internal MOS command with an alias, but have the alias make sure it uses the original command.  NB unlike other commands, no space is required after the `%` character.

This feature was added in MOS 3.0.

## `.` (dot)

Syntax: `*. [-l] [-a] [-s] [-v] [<path>]` (Alias for `Cat`)

This command is an alias for the [`Cat`](#cat) command.  Please see the `Cat` command for more information on how this command works and the options it supports.

## `Cat`

Syntax: `*Cat [-l] [-a] [-s] [-v] [<path>]` (Aliases include `Dir` and `.`)

The "Catalogue" command, which displays a directory listing of the current directory, or of the directory at the path given.

Before MOS 2.2.0 this command did not support any flags and would always show a long-form directory listing of all files in a directory, which includes the file modification date/time, file size and file name.

From MOS 2.2.0 onwards, by default a short listing is shown.  MOS 3.0 will, by default, hide hidden and system files from directory listings.

The flags supported by the `Cat` command are as follows:

- `-l`: Display the long form of the directory listing, which includes the file size and date/time of last modification, introduced in MOS 2.2.0
- `-a`: Show hidden files, introduced in MOS 3.0
- `-s`: Show system files, introduced in MOS 3.0
- `-v`: Hide volume information, introduced in MOS 3.0

When running MOS 3.0 with VDP 2.14.0 or later, the output of the `Cat` command and its various aliases can be [automatically paged](./System-Variables.md#autopaged).  When auto-paging is enabled and the output is longer than the screen output will pause at the end of a page, and you will need to press the "shift" key to continue.

## `CD`

Syntax: `*CD <path>`

Change current directory.

The path provided to the `CD` command uses `/` as the directory separator, and can be either an absolute path or a relative path.  The root directory is `/`.  `CD ..` can be used to move up one directory level.

## `CDir`

Syntax: `*CDir <path>`

This command is an alias for the `CD` command.

Available from MOS 2.2.0 onwards.

## `CLS`

Syntax: `*CLS`

Clear the screen.

(This command performs a `VDU 12`, which is the same as the `CLS` command in BBC BASIC.)

## `Copy`

Syntax: `*Copy <source> <destination>`

Create a copy of a file.

From MOS 2.2.0 onwards, the `Copy` command will also support the use of wildcards in the source file name, such as `*.txt`, and can support the destination being a directory.  It should be noted that the destination path cannot include wildcards.

## `CP`

Syntax: `*CP <source> <destination>`

This command is an alias for the `Copy` command.

Available from MOS 2.2.0 onwards.

## `Credits`

Syntax: `*Credits`

Outputs some credits and version numbers for third-party libraries used in the Agon firmware.

## `Delete`

Syntax: `*Delete [-f] <filename>` (Aliases include `Erase` and `RM`)

Delete a file or folder (must be empty).

As of Console8 VDP 2.2.0 this command supports wild-cards in the filename.  Additionally when deleting a file a confirmation step is required, but if the `-f` flag is provided then the command will not prompt for confirmation before deleting the file.

## `Dir`

Syntax: `*Dir [-l] [-a] [-s] [-v] [<path>]`

This command is an alias for the [`Cat`](#cat) command.  Please see the `Cat` command for more information on how this command works and the options it supports.

## `Do`

Syntax: `*Do <command>`

Execute a command.  The command string used here will be sent through the GSTrans transformation process before use, expanding any [system variables](System-Variables.md) that may be present in the arguments.  One use of this would be to allow for a variable to define which command to execute, which is not usually possible with the command interpreter.  Another use is to allow for commands that do not expand variables by default to include variable values, such as the [`PrintF`](#printf) command.

This command was added in MOS 3.0.

## `Echo`

Syntax: `*Echo <text>`

Prints the given text to the screen.  Before outputting the text is run through a transformation process which allows for the full use of the VDP's control codes.  If you are using MOS 3.0 or later, then you can also use [system variables](System-Variables.md) in the text.

Text will be read one character at a time, and if a translation token is found then the text will be transformed accordingly.  The following tokens are supported:

| Token | Replaced by |
|-------|-------------|
| `|"`  | `"`        |
| `|<`  | `<`        |
| `||` | `|`       |
| `|?`  | Character 127 (A deleting backspace)  |
| `|!`  | Forces top-bit of next character to be set   |
| `|char` | Ctrl (ASCII (uppercase(char)) - 64) |
| `<num>`  | Converts num to character |
| `<variable_name>` | Replaced with the value of the system variable `variable_name` (added in MOS 3) |

The `|char` and `<num>` tokens provide a simple way to send "control codes" to the VDP.

The `|char` token works with a single character, which will be replaced with the ASCII value of the uppercase character minus 64.  This means that a value of `|A` or `|a` will be replaced with character 1, `|B` or `|b` will be replaced with character 2, and so on, equivalent to performing a `VDU 1` or `VDU 2` respectively for `|a` and `|b`.  This provides a simple way to insert control codes into your text.  For example using `|M|J` will send characters 13 and 10, a carriage return and a line feed.  Using `|g` will send a "bell" character and make a beep sound.

Similarly the `<num>` token will be replaced with the character represented by the number given.  For example using `|65` will send the character `A`, and `|13` will send a carriage return.  The number conversion however will support various formats of number.  You can specify a specific base to be used for a number, so `<2_111>` specifies using number 111 in base 2, which will equate to 7 in decimal, and thus be the same as `|g` and make a beeping sound.  Similarly `<3_21>` also produces a beep.  Bases up to 36 are supported, with "digits" in bases over 10 using letters - the most common base over 10 would be base 16, hexadecimal.  You can also specify a hexadecimal number using the `&` prefix, so `<&41>` will also send the character `A`.

These options are essentially just VDU commands that are being sent to the VDP - you should refer to the VDP documentation for all the commands available.

Incomplete tokens, such as ending a string with `|` or `|!` will result in a `Bad string` error.

When using MOS 2.3.0, if you had an invalid number inside `<` and `>` characters then the token would be ignored.  In MOS 3.0 the string between the `<` and `>` characters will be treated as a system variable name, and replaced with the value of that system variable.  If the variable does not exist, then the token will be ignored.

Numeric system variables will be converted to text for output, and macro variables will be evaluated and expanded before being output.

This command was added in MOS 2.3.0, and support for system variables added in MOS 3.0.

## `Erase`

Syntax: `*Erase [-f] <filename>`

This command is an alias for the [`Delete`](#delete) command.

## `Exec`

Syntax: `*Exec <filename>`

Executes commands from a text file.

Commands will be executed one at a time.  If an error occurs processing a command, execution will stop and the error will be reported.

(This is essentially how the `autoexec.txt` file is processed at boot time.)

NB: This command is not available in Quark 1.04.

## `Help`

Syntax: `*Help [<command> | all]`

Displays help information for a command.  If no command is provided, a list of available commands will be displayed.

NB prior to MOS 2.2.0 the `Help` command required a command name to be provided, or the `all` keyword to display all commands.

As of MOS 3.0, the `Help` command will accept a space-separated list of commands to show help for, and will also match commands abreviated with a `.` in the same manner as the command line.  `*Help c.` for example would show help for all commands beginning with `C`.

When running MOS 3.0 with VDP 2.14.0 or later, the output of the `Help` command can be [automatically paged](./System-Variables.md#autopaged).  When auto-paging is enabled and the output is longer than the screen output will pause at the end of a page, and you will need to press the "shift" key to continue.

## `Hotkey`

Syntax: `*Hotkey [<n> [<command string>]]`

Sets or clears programmable function keys.

Using this command you can set the function keys `F1-F12` on your to perform a specific command.  There are three ways in which this command works:

1. If no parameters are provided, then the current hotkeys will be displayed.
2. If a function key number `n` is provided alone without a command string, then the current command assigned to that function key will be cleared.
3. If a function key number `n` and a command string are provided, then the command string will be assigned to that function key.
    - If the command string includes `%s` then the current input line will be substituted in place of `%s`
    - MOS 3.0 supports more sophisticated [argument substitution](Argument-Substitution.md) options

Pressing a function key that has a hotkey definition set will result in the current input line being replaced with the defined hotkey string and "return" being automatically pressed.

As of MOS 3.0, hotkeys are stored using [system variables](System-Variables.md#hotkeys), and can be defined using the `Set` or `SetMacro` commands.  Via the use of `Set` or `SetMacro` you can define hotkeys that do not automatically press "return".  For more information how how hotkeys are handled, and suggestions on how to allow a hotkey to trigger multiple commands, see the [hotkey system variable documentation](System-Variables.md#hotkeys).

## `If`

Syntax: `*If <expression> Then <command> [Else <command>]`

Conditionally execute another command depending on the value of an expression.

At the time of writing, `<expression>` can only be a variable name or a number, as there is not yet a full expression engine available.  The expression will evaluate as true if the variable exists and is either non-zero (for numeric variables) or not an empty string.

The absence of a full expression engine in MOS means that it is not currently possible to check only for the non-existence of a variable as you must always include a `then` command.  There are a few ways to work around this limitation.  One example would be `*if MyVar then try invalid-command else echo MyVar does not exist`, although this would set Try related system variables.  An alternative approach would be `*if MyVar then set MyVar <MyVar> else echo MyVar does not exist`, which would just set MyVar to its current value if it already existed.

Support for this command was added in MOS 3.0.

## `IfThere`

Syntax: `*IfThere <filename> Then <command> [Else <command>]`

Conditionally executes another command depending on the presence of a file or directory.

Support for this command was added in MOS 3.0.

## `JMP`

Syntax: `*JMP <address>`

Jumps to the specified address in memory.  Using this command, you can jump to any address in memory, including the ROM.  If used from the MOS command line, the processor will be in the eZ80 ADL mode.

## `Load`

Syntax: `*Load <filename> [<address>]`

Load a file from the SD card into the specified address in memory.  If no address parameters is specified, then the address will default to `&40000`.

## `LoadFile`

Syntax: `*LoadFile <filename> [<arguments>]`

This command will load a file in a manner appropriate to the file type.  MOS looks up the "load type" information for the file based on its file name extension using a matching system variable, and will then execute the appropriate command to load the file.  If the file type is not recognised, then you will see an "Invalid command" error.

For more information see the documentation for the [corresponding system variables](System-Variables.md#file-type-variables).

This command was added in MOS 3.0.

## `LS`

Syntax: `*LS [-l] [-a] [-s] [-v] [<path>]`

This command is an alias for the [`Cat`](#cat) command.  Please see the `Cat` command for more information on how this command works and the options it supports.

## `Mem`

Syntax: `*Mem`

Displays information on MOS memory usage.

## `MkDir`

Syntax: `*MkDir <name>`

Make a new directory on the SD card.

## `Mount`

Syntax: `*Mount`

Mount the SD card.

If you have ejected your SD card, you can use this command to re-mount it.

## `Move`

Syntax: `*Move <source> <destination>`

This command is an alias for the `RENAME` command.

## `MV`

Syntax: `*MV <source> <destination>`

This command is an alias for the `RENAME` command, and is only available from MOS 2.2.0 onwards.

## `Obey`

Syntax: `*Obey [-v] <filename> [<arguments>]`

Runs an Obey (script) file. The `Obey` command works in a similar manner to [`Exec`](#exec) but offers a few more features. Firstly running a script file with `obey` will set a system variable `Obey$Dir` to reflect the directory in which the obey file is located. Secondly lines in an Obey file will have [argument substitution](Argument-Substitution.md) run on them before they are executed.

The `-v` flag makes the Obey file execute in "verbose" mode, printing each line out before it is executed.

As with `exec`, if a command fails in an obey file then the file execution will stop and the error from that command reported.

By convention, an Obey file should use the file extension `.obey`.

Support for this command was added in MOS 3.0.

## `PrintF`

Syntax: `*PrintF <string>`

Prints the given string to the screen.  This command is similar to the `Echo` command, but supports a subset of unix-style escape transformations for the string, does not expand system variables by default, and does not automatically add a newline to the printed output.  Support for this command was added in MOS 2.3.0.

The following escape sequences are supported:

| Sequence | Replaced by |
|----------|-------------|
| `\f`     | Form feed |
| `\n`     | New line |
| `\r`     | Carriage return |
| `\t`     | Horizontal tab |
| `\\`     | Backslash |
| `\xhh`   | Hexadecimal value |

If an invalid escape sequence is found, it will be ignored/skipped.

Please note that if you wish to use the `PrintF` command with [system variable](System-Variables.md) values then you can run this command via the [`do`](#do) command.  For example `*PrintF <Sys$Time>` would normally just output the string `<Sys$Time>`, but as the `do` command expands any variables before processing the resultant command `*do PrintF <Sys$Time>` would print the current time.

## `Rename`

Syntax: `*Rename <source> <destination>` (Aliases include `MOVE`)

This is the command to rename a file or move a file to a different folder.

To rename a file in the same folder.

`*Rename autoexec.txt autoexec.bak`

Rename a file and move to a different folder (the destination folder must exist).

`*Rename test.bas archive/test.bas`

From MOS 2.2.0 onwards, the `Rename` command also supports allowing the destination to specify a directory.  If the destination is a directory, the source file will be moved to that directory and keep the same filename.  Additionally if the destination is a directory then the source file specified can include a wildcard, such as `*.txt`.

As of MOS 3.0, the `Rename` command supports the use of [system variables](System-Variables.md) and [custom file paths](System-Variables.md#path-variables) inside both the source and destination file names.

## `RM`

Syntax: `*RM [-f] <filename>`

This command is an alias for the [`Delete`](#delete) command, and is only available from MOS 2.2.0 onwards.

## `Run`

Syntax: `*Run [<address | .> [<parameters>]]`

The `Run` command will call an executable binary loaded in memory. If no parameters are passed, then the address will default to `&040000`.  The address parameter can also be replaced with `.` to indicate that the default address of `&040000` should be used.  Any additional parameters will be passed through to the executable.

When using the `Run` command, MOS will check for the presence of a valid [executable](./Executables.md) at the given address by looking for valid header.  If the header header is not found, or is invalid, then it will return an "Invalid executable" error.

More information about parameters that will be passed to an executable when it is run can be found in the [executable documentation](./Executables.md#parameters).

## `RunBin`

Syntax: `*RunBin <filename> [<arguments>]`

This command will load and run a binary file. It differs from performing a `load` followed by a `run` in that it will work out the appropriate memory address to load the file into.  This is done by comparing the path to the given file with the current [`Moslet$Path` system variable](System-Variables.md#system-path-variables), so moslets will get loaded and run at the appropriate moslet memory location.

This command was added in MOS 3.0.

## `RunFile`

Syntax: `*RunFile <filename> [<arguments>]`

This command will run a file in a manner appropriate to the file type.  MOS looks up the "run type" information for the file based on its file name extension using a matching system variable, and will then execute the appropriate command to run the file.  If the file type is not recognised, then you will see an "Invalid command" error.

For more information see the documentation for the [corresponding system variables](System-Variables.md#file-type-variables).

This command was added in MOS 3.0.

## `Save`

Syntax: `*Save <filename> <address> <size>`

Save a block of memory to the SD card.

## `Set`

Syntax: `*Set <option> <value>`
Syntax: `*Set <variable-name> <value>` (MOS 3.0 or later)

Set a system option, or a [system variable](System-Variables.md).

NB before MOS 2.2.0 the `Set` command was case-sensitive for the option name.

On MOS 2.3.x or earlier this command would only support two system options, `KEYBOARD` and `CONSOLE`, as documented below.

### Keyboard Layout

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

\* These layouts are only supported from MOS 2.2.0 onwards.  The exact keyboard layouts supported will depend on the version of the VDP firmware you have installed.  Prior to MOS 2.2.0 keyboard values of 9 or above would be ignored.  From MOS 2.2.0 onwards any keyboard value can be specified - if the keyboard layout is not supported by the VDP firmware then it will be ignored.

### Console Mode

Syntax: `*SET CONSOLE <n>`

Sets the console mode, where if `n` is `0` then console mode will be disabled, and if `n` is `1` then console mode will be enabled.

When console mode is enabled, all (initial) "command" and text bytes sent to the VDP will be echoed out of the VDP to any attached USB serial device.  This can be useful for debugging purposes.

It is recommended that instead of using this command, if you are running Console8 VDP 2.5.0 or later, you should use `VDU 2` instead to enable the "printer", which also outputs from the VDP to the same serial device.  The main difference is that the "printer" device will filter out any control codes, providing a safer output, whereas the console mode will output everything.  The printer can be disabled with `VDU 3`.  Additionally pressing `CTRL+B` on the keyboard will enable the printer, `CTRL+C` will disable it, and `CTRL+P` will toggle it.

### System Variables

From MOS 3.0 onwards, the `Set` command can also be used to set [system variables](System-Variables.md).  The syntax for setting a system variable is `*Set <variable-name> <value>`.

When you use this command a string-type system variable of the given name will either be set or changed to the given value.

Some variables provided by MOS are "code" type variables, which means that they are not just a simple string, but have some special behaviour.  Examples of these are the `Sys$Date`, `Sys$Time` and `Sys$Year` variables which expose the real-time clock as system variables.  Attempting to set either the time or date variable will adjust the real-time clock on the system.  The prior uses of the `Set` command for `keyboard` and `console` are both supported via "code" variables, so existing scripts that use those commands will continue to work.

It should be noted that the "value" part of this command is transformed when the variable is set, so for example `Set TimeNow The time is <sys$time>` will set the `TimeNow` variable to a string with the time when the command was executed.  Thus if you then did an `echo <timenow>` you would see the time when the `set` command was run.  To avoid this problem you should use the `SetMacro` command instead.

## `SetEval`

Syntax: `*SetEval <variable-name> <expression>`

This command evaluates an expression and sets the named variable to the result.

As of when this document was written, MOS does not yet have a full expression engine.  For now, when you use `SetEval` it will create (or replace) the named variable with the result of the expression.  If the expression is a number, or refers to a number variable, then the resultant variable will be a "number" type variable of the value given.  If the expression is a string (or a macro) then the new variable will be a "string" type variable copy of the named variable.

This command was added in MOS 3.0.

## `SetMacro`

Syntax: `*SetMacro <variable-name> <value>`

This works in a similar manner to the `set` command but will set a "macro" style variable.  With a macro, the variable value is not pre-transformed when the variable is set, but instead will get transformed when the variable is used.  This means that `SetMacro TimeNow The time is <sys$time>` would mean that a command to `echo <TimeNow>` would show the time when the `echo` command is executed.

This command was added in MOS 3.0.

## `Show`

Syntax: `*Show [<variable-name>]`

Displays the value of a system variable matching the given name.

The name given can use wildcards, specifically `*` to match any number of characters, and `#` to match a single character.  All variables matching the given pattern will be shown.  If no variable name is provided, then all system variables will be displayed.

This command was added in MOS 3.0.

When running MOS 3.0 with VDP 2.14.0 or later, the output of the `Show` command can be [automatically paged](./System-Variables.md#autopaged).  When auto-paging is enabled and the output is longer than the screen output will pause at the end of a page, and you will need to press the "shift" key to continue.

## `Time`

Syntax: `*Time [<YYYY> <MM> <DD> <hh> <mm> <ss>]`

Set and read the ESP32 real-time clock.

When no parameters are provided, the current date and time will be displayed.  Setting the real-time clock requires a complete set of parameters, including the year, month, day, hour, minute and second.  

It should be noted that the Agon platform has no concept of time zones, or daylight savings time, so you should consider the time set as your local time.

## `Try`

Syntax: `*Try <command>`

`Try` will perform the given command, trapping any errors that may occur.  It will set `Try$ReturnCode` with the return code value for the command, and if an error occurred it will set `Try$Error` to the corresponding error string.

The `Try` command allows for commands that can fail to be included in script files that are run using `exec` or `obey` as they allow for the execution of the script to continue.

This command was added in MOS 3.0.

## `Type`

Syntax: `*Type <filename>`

Display the contents of a text file on the screen.

From MOS 3.0 onwards the `Type` command is tolerant of control characters in the file, and will display them by "escaping" them, showing them as a two character combination of a `|` character followed by a letter corresponding to the control character.  For example, a carriage return will be displayed as `|M`, and a line feed as `|J`.  This is similar to how the `echo` command works.  Prior to MOS 3.0 the `Type` command would not check for control characters and just output the file as-is to the screen which could cause unexpected results.

When running MOS 3.0.1 with VDP 2.14.0 or later, the output of the `Type` command can be [automatically paged](./System-Variables.md#autopaged).  When auto-paging is enabled and the output is longer than the screen output will pause at the end of a page, and you will need to press the "shift" key to continue.

## `Unset`

Syntax: `*Unset <variable-name>`

Unsets a system variable matching the given variable name.  This will work for any variable set, except for code variables which cannot be unset.  As with the `show` command, the variable name given can include wildcards, in which case all variables that match the given name pattern will be removed.

## `VDU`

Syntax: `*VDU <char1> <char2> ... <charN>`

Write a stream of character bytes to the VDP.  This can be used to perform various control functions, such as clearing the screen, changing the screen mode, setting the cursor position, or changing the text colour.  More information on VDU commands can be found in the [VDP documentation](../VDP.md).

This command is similar to the `VDU` command in BBC BASIC, but instead of separating arguments with commas you must instead separate them with spaces.

Prior to MOS 2.2.0, the `VDU` command would only accept 8-bit numbers as arguments.  Any values provided greater than 255 would be ignored.

From MOS 2.2.0 onwards this command will now also support 16-bit numbers as arguments, and several different ways to indicate hexadecimal numbers.  The BBC BASIC standard of suffixing a number with a semicolon character `;` to indicate a 16-bit number is supported.  As well as the `&` prefix for hexadecimal numbers, you can also use `0x` as a prefix, or `h` as a suffix.  Hexadecimal numbers provided greater than 255 will always be treated as 16-bit numbers.

As noted above, from MOS 3.0 on numbers can be provided in various other formats, which is supported by the `VDU` command.


