# MOS Executables

Executable programs for the Agon platform have, to date, conventionally used a `.bin` file extension.  In principle however any file can be used as an executable.

Essentially the process for running any executable is the same.  The following three steps will be performed:

1. The program file will be opened and read into memory
2. It will then will be checked to see if it is a valid executable
3. If it is valid, then the program will be run

The exact details of these steps may differ and vary slightly depending on the version of MOS you are using, but essentially these steps broadly remain the same.

MOS essentially supports two different types of programs; ["moslets"](../MOS.md#moslets) and regular programs.

Moslets are always built to be loaded and run at address `0x0B0000`, the start of the "moslet" memory space in the Agon platform's [memory map](../MOS.md#memory-map), and should be smaller than 32 kilobytes in size to fit into that space.

Regular prograns are usually built to be loaded and run at address `0x040000`, the start of the "user RAM" area of memory.  This is the address that MOS will automatically use when loading and running a program.

MOS 2.3 and later will additionally check to ensure that a program cannot be loaded into memory that will overlap the MOS system area of memory.

## Parameters passed to executables {#parameters}

When a program is run, whether that is through the use of the [`Run` command](Star-Commands.md#run) or via a different mechanism, MOS will set up the following processor registers before the program is entered:

- `A` will be set to the current memory bank value `MB`
- `DE(U)` the execution address
- `HL(U)` pointer to any additional command line parameters

Notes about the processor stack can be [found here](../MOS.md#the-stack).

## The MOS Executable File Format

To facilitate checking for valid executables, MOS uses a simple executable file format.

The basic format requires a header to be present at an offset of 64 bytes into the file, as follows:

| Offset | Size | Contents |
|--------|------|----------|
| 0x40   | 3 bytes | the word `MOS` in ASCII bytes |
| 0x43   | 1 byte | header version |
| 0x44   | 1 byte | executable type: `0` = Z80, `1` = ADL |

You will note that this header format is very simple and essentially includes only one piece of information, whether the executable code should be run in Z80 or ADL mode on the eZ80 processor.  There is no information here about where the program should be loaded into memory, so it is not possible to tell from the header whether a program is a moslet, a regular program, or some other type of program file.

The header version byte should be set to `0` for all programs that were written for MOS 1 and MOS 2.  Up to and including MOS 3.0 the version byte is not actually checked.

### The "advanced" header format {#advanced-header}

As of the MOS 3.0 release an advanced header format is being introduced which allows for the inclusion of additional information in the header.

Please note that this header format is backwards compatible with the original header format, and programs that use this new header format will still be able to run on earlier versions of MOS.

The main reason for this new format is is to prepare for [MOS Modules](./Modules.md).  When the MOS modules system is introduced, executables with a header version of `0` will not be considered to be "module safe", and API calls, commands, and functions provided by modules will not be available to them.

Some APIs, commands, and functions introduced in MOS 3.0 may be moved to modules in the future.  To guarantee that a program using these features will continue to work in the future you should ensure that your program is "module safe" and has an appropriate header.

As previous versions of MOS do not actually check the header version, it is possible to use the new header format in existing programs that can run on all versions of MOS.  As a result you should be aware that any additional information or or settings included in the header will be ignored by previous versions of MOS.  You should not assume that including such information is a guarantee that it will be understood when your program is run.

The new header format is as follows:

| Offset | Size | Contents |
|--------|------|----------|
| 0x40   | 3 bytes | the word `MOS` in ASCII bytes |
| 0x43   | 1 byte | header version (this should be `1` for "advanced") |
| 0x44   | 1 byte | executable type: `0` = Z80, `1` = ADL |
| 0x45   | 1 byte | flags |
| 0x46   | 1 byte | bit-inverted copy of the flags byte at offset `0x45` |
| 0x47   | 3 bytes | optional load/execution address (little-endian) - only use this if the corresponding flag is set |

The flags byte is a bit field.  As the original header version byte was not well documented, and is not actively checked inside earlier versions of MOS, it is possible that some programs may exist that have the header version byte set to `1`.  To avoid any potential false-positives, an inverted copy of the flags byte is included to allow the flags byte to be verified.

The following bits are defined:

| Bit | Meaning | Description |
|-----|---------|-------------|
| 0   | Module safe | The program is "module safe", as it does not use the "moslet" area of memory (where modules are loaded) |
| 1   | Module compatible | The program uses the moslet area of memory, but modules can be loaded into that area of memory.  When this bit is set when a module needs to be loaded MOS will save the moslet area of memory to disk before loading a module and executing the command or API that required it.  On returning from the module, MOS will restore the moslet area of memory from disk.  Programs with this bit set must ensure that any API calls they make will not use the moslet area for data buffers the API calls require. |
| 2   | Strip trailing spaces from arguments | When this bit is set, any trailing spaces in the arguments string passed to the program will be stripped.  Please note that this is the default behaviour for executables with a header version of `0`, but the default behaviour for executables with a header version of `1` is to not strip trailing spaces.  This bit is provided for backwards compatibility with earlier versions of MOS. |
| 3   | Header includes load/execution address | The default address to load and execute this executable is set at offset 0x47-49.  For Z80 executables the most significant byte (at 0x49) will be ignored |
| 4-7 | Reserved | These bit is reserved for future use, and must be set to zero | 

More advice on the meanings of "module safe" and "module compatible" can be found in the [MOS Modules](./Modules.md) documentation.

Future versions of this header version may include support for more bits in the flags byte to indicate that further information is provided.  When support for new flags is added unless there is a good reason to do so the header version will not be changed and remain `1`.

It should also be noted that whilst this new header format has been introduced with the release of MOS 3.0, the MOS 3.0 release does not understand the new header extensions, and will ignore them.  They are included and documented here to allow developers to start using them in their programs in preparation for future versions of MOS.

It is likely that the next release of MOS will understand this new header format and at least provide support for bits 2 and 3 in the flags byte.  When MOS Modules are introduced, bits 0 and bit 1 will be supported.
