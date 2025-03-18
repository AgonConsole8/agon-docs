# Argument substitution

Several features in MOS have support for "argument substitution".  This includes how the system handles [aliases](System-Variables.md#command-aliases), [hotkeys](System-Variables.md#hotkeys), [loading/running files with given file extensions](System-Variables.md#file-type-variables), and [running Obey files](Star-Commands.md#obey).

Argument substitution is used by MOS as a way to insert arguments into a new command.  Essentially, a source string is defined which includes placeholder tokens for arguments to be inserted into.  When the command is executed, the system will replace these tokens with the actual arguments that were passed to the command.

Before MOS 3 the only command that supported argument substitution was the [`Hotkey` command](Star-Commands.md#hotkey).  It would only support a single form of substitution token in the given command string in the form of `%s`, which would insert the whole of the current input line at that point.  This has changed in MOS 3 where the system now supports a more flexible form of argument substitution.

From MOS 3 onwards, arguments substitution is done by using the `%` character followed by a number from 0-9 to specify which argument to insert.  These are arguments are taken from the original command.  For instance, `%0` will be replaced by the first argument to the command, `%1` will be replaced by the second argument, and so on.  If you want to include a literal `%` character in the command, you can escape it by using `%%`.  You can also use `%*<n>` to insert all arguments starting from a particular numbered argument into the command.  For example, `%*2` will insert all arguments from the second argument onwards.  For backwards compatibility, `%s` is still supported, and is equivalent to `%*0`.

Arguments can be wrapped with double-quote characters `"` to allow for arguments that contain spaces.

Arguments may also be used multiple times in the command string.  For example, the command `*set Alias$DoubleEcho echo %0 %0` will create a command alias called `DoubleEcho` that will echo the first argument twice, so performing the command `*DoubleEcho hello` would produce the output `hello hello`.

Command aliases, including file load/run types, will automatically append any arguments beyond the last used argument to the end of the resultant command.

Creating a command alias with the name of an existing command allows you to override that command, changing how it works.  The original command can still be used by prefixing the command with a `%` character, as prefixing a command with `%` causes alias-resolution to be skipped.  An example of where you might wish to do this is to set the `Dir` command to display the directory by default in long format, rather than short form.  This can be achieved with `*set Alias$Dir %dir -l %0`.  If you omit the `%` character here before the `dir` command, the system will attempt to recursively call the alias, and you will get a `Too deep` error.

Prefixing aliases with the `%` character to skip alias resolution can be useful in script files to help ensure that you are definitely using the original command, rather than an alias that another script may have set up.

The argument substitution system in MOS 3 is available for use by programs through the [`mos_substituteargs` API call](../MOS-API.md#0x35-mos_substituteargs).  Whilst this is commonly used to construct commands, it can be used for other purposes too.  At its core, argument substitution is a simple text replacement system, and can be used for any purpose where you need to insert text into a string.  It takes a template string, a separate string for the arguments, and a buffer to write the resultant string to where placeholders in the template string have been replaced with sub-strings taken from the arguments.
