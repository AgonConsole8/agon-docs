# Argument substitution

Some features in MOS will support "argument substitution".  This includes how the system handles [aliases](System-Variables.md#command-aliases), [hotkeys](System-Variables.md#hotkeys), [loading/running files with given file extensions](System-Variables.md#file-type-variables), and [running Obey files](Star-Commands.md#obey).

Argument substitution is a way to insert arguments into a new command.

Before MOS 3 the only command that supported argument substitution was the [`Hotkey` command](Star-Commands.md#hotkey).  It would only support a single form of substitution token in the given command string in the form of `%s`, which would insert the whole of the current input line at that point.  This has changed in MOS 3, and now the system supports a more flexible form of argument substitution.

From MOS 3 onwards, argument substitution is done by using the `%` character followed by a number to specify which argument to insert, numbered from zero.  These are arguments are taken from the original command.  For instance, `%0` will be replaced by the first argument to the command, `%1` will be replaced by the second argument, and so on.  If you want to include a literal `%` character in the command, you can escape it by using `%%`.  You can also use `%*<n>` to insert all arguments starting from a particular numbered argument into the command.  For example, `%*2` will insert all arguments from the second argument onwards.

Command aliases, including file load/run types, will automatically append any arguments beyond the last used argument to the end of the resultant command.

Creating a command alias with the name of an existing command allows you to override that command, changing how it works.  The original command can still be used by prefixing the command with a `%` character, as prefixing a command with `%` causes alias-resolution to be skipped.  An example of where you might wish to do this is to set the `Dir` command to display the directory by default in long format, rather than short form.  This can be achieved with `*set Alias$Dir %dir -l %0`.  If you omit the `%` character here before the `dir` command, the system will attempt to recursively call the alias, and you will get a `Too deep` error.

Prefixing aliases with the `%` character to skip alias resolution can be useful in script files to help ensure that you are definitely using the original command, rather than an alias that another script may have set up.

