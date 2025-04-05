# VDP Variables

VDP variables provide a way to both read and change the state of the VDP.  They can be used to enable an experimental feature, read the current state of the VDP, or change the state of the VDP.

VDU variables contain 16-bit values.  Some variables may only use the lower 8 bits.

Variables are currently split into three general categories: test flags, system settings, and VDU variables.  In the future we will add more variables to expose the state of the audio system.  These categorites are given some broad ranges of variable IDs to allow for future expansion, and to allow for easy identification of the type of variable.

Test flags are used to enable a feature that may either be experimental, not yet fully implemented, have an API that might change, or not fully tested.

System settings variables provide access to VDP system information, such as memory usage, real-time clock data, and keyboard settings.

VDU variables provide access to the graphics system state, including context-specific information that may change when switching [context](Context-Management-API.md).

The VDP variables system was added in Console8 VDP 2.9.0, and was initially used only for test flags.

The Console8 VDP 2.12.0 release has added many new variables that expose the state of the VDP, and allow for changes to be made to that state.  It is also now possible to use VDP variables in buffered commands.  This allows for conditional commands to be used to check against a variable, and the contents of a variable to be read into a buffer.


## Variable APIs

There are just two API calls to directly work with VDP variables, one to set a variable, and another to clear it.  As of VDP 2.12.0 the buffered command API also allows for the reading of variables into a buffer, and for conditional commands to be used to perform their checks against a variable.

The commands to set and clear variables are documented in the [system commands](System-Commands.md) documentation.  Briefly they are:

* `VDU 23, 0, &F8, variableId; value;`: Set a VDP Variable
* `VDU 23, 0, &F9, variableId;`: Clear a VDP Variable

The buffered command API is documented in the [buffered commands](Buffered-Commands-API.md) documentation.  The "conditional" commands there are now extended to allow for the use of variables, and an additional command to read the value of a variable and store it into a buffer has been added.


## Variable ID ranges

All variables listed are available from VDP 2.12.0 onwards.  Variables that were introduced in 2.9.0 and 2.11.0 are marked as such.

The broad ranges of variable IDs are as follows:

| Variable ID range | Description |
| ----------- | ----------- |
| &1-&FF | Test flags |
| &100-&FFF | System settings |
| &1000-&1FFF | VDU variables (graphics/text system) |
| &2000-&2FFF | Reserved for audio system variables |
| &3000-&3FFF | Reserved for future use |
| &4000-&FFFF | Available for general use |

The system will not prevent you from using variables in reserved ranges, although inside the "VDU variables" range values that the system does not directly support will not be stored, and cannot be read.


## Test Flags

As of Console8 VDP 2.9.0 the following test flags are supported:

| Variable ID | Value | Description |
| ------- | ----- | ----------- |
| 1 | 0 (any) | Enable the Affine Transforms feature (available from 2.9.0) |
| 2 | 0 (any) | Enable hardware sprites |

If a flag is set that is not recognised then it will have no effect.  This means that if a feature graduates from being a test feature and no longer requires a flag to be set for use then, so long as the API for the feature remains the same, software that sets the flag to enable the feature will still work.

For the current test flags that the VDP supports, any value can be set to enable the feature.  Future flags may require specific values to be set to control how the feature works.


## System Variables

Broadly speaking, system variables are used to read and change the state and configuration of the VDP that are not _directly_ related to the current graphics context state or screen mode.  Within the system variables block there are sub-blocks for different types of system information.

The sub-ranges within system variables are broadly as follows:

| Block | Description |
| ----------- | ----------- |
| &100-&1FF | Communications system settings |
| &200-&208 | Real-time clock data |
| &209-&20F | Reserved for future use |
| &210-&21F | VDP memory information |
| &220-&22F | Keyboard settings |
| &230-&23F | Context management |
| &240-&2FF | Reserved for future use |
| &300-&3FF | Graphical system enhancement settings/flags |
| &400-&4FF | Bitmap/Sprite system control settings |
| &500-&FFF | Reserved for future use |

Many system variables can be set and adjusted by other VDU commands

As of Console8 VDP 2.12.0 the following system variables are supported:

| Variable ID | Value | Read-only | Clearable | Description |
| ------- | ----- | --- | --- | ----------- |
| &0101 | 0/1 | | X | Full duplex UART hardware flow control flag. This is intended for internal use by MOS. NB setting this flag will break communications with MOS unless a suitable version of MOS that supports full duplex flow control.  The first version of MOS to support this is MOS 3.0 alpha 3 |
| &0102 | n/a | | X | Reserved for future use (Buffer size on MOS for VDP protocol packets) |
| &0110 | 0/1 | | X | Reserved for future use (Echo back received data, for redirect/spool, with a suitable version of MOS that supports this feature) |
| &0200 | 0-999 | | | Real-time clock year |
| &0201 | 1-12 | | | Real-time clock month |
| &0202 | 1-31 | | | Real-time clock day |
| &0203 | 0-23 | | | Real-time clock hour |
| &0204 | 0-59 | | | Real-time clock minute |
| &0205 | 0-59 | | | Real-time clock second |
| &0206 | 0-999 | X | | Real-time clock millisecond |
| &0207 | 0-6 | X | | Real-time clock weekday |
| &0208 | 0-366 | X | | Real-time clock day of year |
| &0210 | | X | | Free PSRAM low bytes |
| &0211 | | X | | Free PSRAM high bytes |
| &0212 | | X | | Number of buffers used |
| &0220 | 0-17 | | | Keyboard layout (setting to an invalid number will set to zero) |
| &0221 | 0/1 | | | Control keys on/off (setting to any non-zero value sets to 1) |
| &0230 | 0-255 | | | Current active context ID |
| &0300 | 0 (any) | | X | Tile engine flag (enables layers commands, available from VDP 2.11.0) |
| &0310 | 0 (any) | | X | Enables copper features flag |
| &0400 | 0 (any) | | X | Prefer hardware sprites flag. When set, all sprites will be set to be hardware sprites after calling the "Reset sprites" API, if the "Enable hardware sprites" test flag has also been set |


## VDU Variables

VDU variables are numbered in the range &1000-&1FFF and are used to expose information on the current graphics system state.  This includes both information about the current screen mode, and the current context state.

The set of variables are loosely based on the VDU variables available in Acorn's RISC OS operating system, and where appropriate they are numbered the same.  The Agon VDP includes many extensions to include additional information that is specific to the Agon platform.

All variables within this range are reserved for use by the VDU system.  Any values that are not recognised will not be stored, and cannot be read.

Flag variables use values 0 to indicate disabled, and 1 to indicate enabled.  Setting a flag to any non-zero value is counted as if setting to 1.

Some variables provide coordinates.  For those variables that are marked as "screen coordinates", the origin is at the top-left of the screen, and the location is measured in pixels.  For those variables that are marked as "character coordinates", the origin is at the top-left of the screen, and the location is measured in characters.  Variables shown as "OS coordinates" will reflect the currently selected coordinate system, as defined in variable &1057.  When the default coordinate system is selected, the origin is at the bottom-left of the screen, and the location is measured in OS units, where the screen is defined as 0-1279 for X and 0-1023 for Y.

| Variable ID | Value | Read-only | Clearable | Description |
| ------- | ----- | --- | --- | ----------- |
| 0x1001 | | X | | Text columns - (characters) |
| 0x1002 | | X | | Text rows - (characters) |
| 0x1003 | 1/3/15/63 | X | | Max logical colour number for current screen mode |
| 0x100B | | X | | Screen width in pixels - 1 |
| 0x100C | | X | | Screen height in pixels - 1 |
| 0x100D | 1/2 | X | | Number of screen banks (1 for single-buffered modes, 2 for double-buffered) |
| 0x1017 | 0-255 | | | Current line thickness (pixels) |
| 0x1018 | | | | Text cursor, absolute X position (chars) |
| 0x1019 | | | | Text cursor, absolute Y position (chars) |
| 0x1020 | | | | Frame counter low word (the frame counter is a 32-bit value) |
| 0x1021 | | | | Frame counter high word |
| 0x1055 | | X | | Current screen mode number |
| 0x1056 | 0/1 | | | Legacy modes flag |
| 0x1057 | 0/1 | | | Coordinate system (0 = screen/pixel coordinates, 1 = logical/OS (default)) |
| 0x1058 | 0/1 | | | Paged mode flag |
| 0x1066 | 0-255 | | | Cursor behaviour flags byte, as set via VDU 23,16,x,y |
| 0x1067 | 0/1 | | | Text cursor visibility |
| 0x1068 | | | | Text cursor block horizontal start column |
| 0x1069 | | | | Text cursor block horizontal end column |
| 0x106A | 0-31 | | | Text cursor block vertical start row |
| 0x106B | | | | Text cursor block vertical end row |
| 0x106C | 0-3 | | | Text cursor appearance, write only (0 = steady, 1 = off, 2 = fast blink, 3 = slow blink) |
| 0x1070 | | X | | Active cursor type (0 = Text cursor, 1 = Graphics cursor) |
| 0x1080 | | | | Graphics window, LH column, screen coordinates |
| 0x1081 | | | | Graphics window, Bottom row, screen coordinates |
| 0x1082 | | | | Graphics window, RH column, screen coordinates |
| 0x1083 | | | | Graphics window, Top row, screen coordinates |
| 0x1084 | | | | Text window, LH column, character coordinates |
| 0x1085 | | | | Text window, Bottom row, character coordinates |
| 0x1086 | | | | Text window, RH column, character coordinates |
| 0x1087 | | | | Text window, Top row, character coordinates |
| 0x1088 | | | | Graphics origin, X, OS coordinates |
| 0x1089 | | | | Graphics origin, Y, OS coordinates |
| 0x108A | | | | Graphics cursor, X, OS coordinates |
| 0x108B | | | | Graphics cursor, Y, OS coordinates |
| 0x108C | | | | Oldest Graphics cursor, X, screen coordinates |
| 0x108D | | | | Oldest Graphics cursor, Y, screen coordinates |
| 0x108E | | | | Previous Graphics cursor, X, screen coordinates |
| 0x108F | | | | Previous Graphics cursor, Y, screen coordinates |
| 0x1090 | | | | Graphics cursor, X, screen coordinates |
| 0x1091 | | | | Graphics cursor, Y, screen coordinates |
| 0x1097 | 0-7 | | | GCOL action for foreground colour |
| 0x1098 | 0-7 | | | GCOL action for background colour |
| 0x1099 | 0-63 | | | Graphics foreground (logical) colour |
| 0x109A | 0-63 | | | Graphics background (logical) colour |
| 0x109B | 0-63 | | | Text foreground (logical) colour |
| 0x109C | 0-63 | | | Text background (logical) colour |
| 0x10A1 | | X | | Max mode number (not double-buffered) |
| 0x10A2 | | X | | X font size, graphics cursor |
| 0x10A3 | | X | | Y font size, graphics cursor |
| 0x10A4 | | X | | X font spacing, graphics cursor |
| 0x10A5 | | X | | Y font spacing, graphics cursor |
| 0x10A7 | | X | | X font size, text cursor |
| 0x10A8 | | X | | Y font size, text cursor |
| 0x10A9 | | X | | X font spacing, text cursor |
| 0x10AA | | X | | Y font spacing, text cursor |
| 0x10F2 | | | | Dotted line pattern length |
| 0x10F3 | | | | Line pattern, bytes 0-1 |
| 0x10F4 | | | | Line pattern, bytes 2-3 |
| 0x10F5 | | | | Line pattern, bytes 4-5 |
| 0x10F6 | | | | Line pattern, bytes 6-7 |
| 0x1100 | | | | Width of text window in chars |
| 0x1101 | | | | Height of text window in chars |
| 0x1118 | | | | X position of text cursor within text window |
| 0x1119 | | | | Y position of text cursor within text window |
| 0x111A | | | | X position of text cursor in screen coordinates |
| 0x111B | | | | Y position of text cursor in screen coordinates |
| 0x1200-0x123F | 0-63 | | | Palette entries.  Maps logical colours to physical screen colours. The entries used will depend on the number of colours in the current screen mode |
| 0x1300-0x13FF | | | | Character to bitmap mapping.  Value is a 16-bit bitmap ID, or 65535 if character is not mapped.  See `VDU 23, 0, &92, char, bitmapId;` |
| 0x1400 | | | | Currently selected bitmap ID (16-bit bitmap ID) |
| 0x1401 | | X | | Count of bitmaps used |
| 0x1402 | | | | Current bitmap transform ID. Must be set to a buffer ID containing a valid affine transform.  The affine transforms flag must be set to change this value |
| 0x1410 | 0-255 | | | Current sprite ID |
| 0x1411 | | X | | Number of sprites active (not necessarily visible) |
| 0x1420 | | | | Mouse cursor ID |
| 0x1441 | 0/1 | | | Mouse cursor enabled. Mouse data can only be read if the mouse is enabled |
| 0x1442 | | | | Mouse cursor X position in screen coordinates |
| 0x1443 | | | | Mouse cursor Y position in screen coordinates |
| 0x1444 | 0-7 | X | | Mouse cursor button status.  Bit 0 indicates left button pressed, bit 1 the right button, and bit 2 the middle button |
| 0x1445 | | X | | Mouse wheel delta |
| 0x1446 | | | | Mouse sample rate |
| 0x1447 | | | | Mouse resolution |
| 0x1448 | | | | Mouse scaling |
| 0x1449 | | | | Mouse acceleration |
| 0x144A | | | | Mouse wheel acceleration |

