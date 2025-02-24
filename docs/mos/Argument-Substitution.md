# Argument substitution

Some features in MOS will support "argument substitution".  This includes how the system handles aliases, hotkeys, and running Obey files.



Aliases support argument substitution.  This is done by using the `%` character followed by a number to specify which argument to insert, numbered from zero.  For instance, `%0` will be replaced by the first argument to the alias, `%1` will be replaced by the second argument, and so on.  If you want to include a literal `%` character in the alias, you can escape it by using `%%`.  Argument substitution can also support placing all arguments from a particular argument using `%*n`, so `%*2` will place all arguments from argument two onwards.  Any arguments beyond the last used argument by the alias will be automatically appended to the end of the resultant command.

Creating an alias with the name of an existing command allows you to override that command, changing how it works.  The original command can still be used by prefixing the command with a `%` character, as prefixing a command with `%` causes alias-resolution to be skipped.  An example of where you might wish to do this is to set the `Dir` command to display the directory by default in long format, rather than short form.  This can be achieved with `*set Alias$Dir %dir -l %0`.  If you omit the `%` character here before the `dir` command, the system will attempt to recursively call the alias, and you will get a `Too deep` error.

