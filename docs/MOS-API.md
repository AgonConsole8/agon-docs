# What is the MOS API

The MOS API can be used by external applications to access MOS functionality.

Please note that this documentation uses assembler in the examples in a format that is compatible with the Zilog ZDS II assembler.  The assembler syntax used in BBC BASIC is similar, but not identical.  Additionally you will need to use the new ADL version of BBC BASIC to use the new eZ80 ADL mode and extended instruction set.

This documentation is not intended as a tutorial on eZ80 assembler, but as a reference for those who are already familiar with the eZ80 or Z80 instruction set and wish to use MOS APIs in their programs.

## Usage from Z80 assembler

There are four RST instructions for accessing MOS functionality from Z80.

- `RST 00h`: Reset the eZ80
- `RST 08h`: Execute a MOS command
- `RST 10h`: Output a single character to the VDP
- `RST 18h`: Output a stream of characters to the VDP (MOS 1.03 or above)
- `RST 38h`: Outputs a crash report (Console8 MOS 2.3.0 or above)

In addition, if you are using the Zilog ZDS II assembler you may wish to include the file `mos_api.inc` in your project.  The Console8 version can be found in the folder [src](https://github.com/AgonConsole8/agon-mos/tree/main/src) of project [agon-mos](https://github.com/AgonConsole8/agon-mos).  The original Quark versions of this file can be found in the folder [src](https://github.com/breakintoprogram/agon-mos/tree/main/src) of project [agon-mos](https://github.com/breakintoprogram/agon-mos).

NB:

- Using the `RST.LIS` opcode in an eZ80 assembler will ensure the MOS RST instructions are called regardless of the eZ80s current addressing mode.
- In the `mos_api.inc` file you will find:
    - EQUs for all the MOS commands, data structures and system variables.
    - An incomplete list of VDP control variables.  For a full list, see the [VDP documentation](VDP.md)
    - A complete list FatFS APIs, however these are not yet all implemented in MOS.  Those that are implemented are documented below.

Further information on the `RST` handlers provided by MOS are as follows:

### `RST 08h`: Execute a MOS command

Parameters:

- `A`: MOS command number to execute

NB:

- There is a macro in `mos_api.inc` with EQUs for all the MOS commands
- Other MOS-command dependant parameters may be required

Macro:

```
;
; Macro for calling the API
; Parameters:
; - `F`unction: One of the function numbers listed above
;
MOSCALL:	MACRO	function
			LD	A, function
			RST.LIS	08h
			ENDMACRO
```

Example:

```
; OSRDCH: Read a character in from the ESP32 keyboard handler
;
OSRDCH:	MOSCALL	mos_getkey
		OR	A
		JR	Z, OSRDCH		; Loop until key is pressed
		RET
```

### `RST 10h`: Output a single character to the VDP

Parameters:

- `A`: Character to output

Example:

```
; OSWRCH: Write a character out to the ESP32 VDU handler via the MOS
; A: Character to write
;
OSWRCH:	RST.LIS	10h			; This calls a RST in the eZ80 address space
		RET
```

### `RST 18h`: Output a stream of characters to the VDP (MOS 1.03 or above)

Parameters:

- `HL(U)`: Address of the data stream (16-bit for Z80 mode, 24-bit for ADL mode)
- `BC`: Length of stream (or 0 if the stream is delimited)
- `A`: Stream delimiter (if `BC`=0)

Returns:

- `A`: Last character displayed (length mode) OR Delimiter (delimiter mode)
- `BC`: 0
- `HL(U)`: Address of last character displayed + 1 (length mode) OR location of delimiter (delimiter mode)
- `E`: Value of `A` upon entry

Example:

```
; Write a stream of characters to the VDP
; HLU: Address of buffer containing data - if in 16-bit segment, U will be replaced by MB
;  BC: Number of characters to write out, or 0 if the data is delimited
;   A: End of data delimiter, i.e. 0 for C strings
;
		LD	HL, text		; Address of text
		LD	BC, 0			; Set to 0, so length ignored...
		LD	A, 0			; Use character in A as delimiter
		RST.LIS	18h			; This calls a RST in the eZ80 address space
		RET
;
text:	DB	"Hello World", 0
```

### `RST 38h`: Outputs a crash report (Console8 MOS 2.3.0 or above)

This command will output a crash report to the screen.  This report will show the current processor state, and the top of the stack.  This can be useful for debugging purposes.

This command works in conjunction with the fact that as of Console8 MOS 2.3.0, on initial startup memory will be reset to contain `0xFF` bytes in every location, which equates to a `RST 38h` instruction.  This means that for many system crashes execution will end up at the `RST 38h` instruction, and a crash report will be displayed on the screen.


## The MOS API

MOS API calls can be executed from a classic 64K Z80 segment or whilst the eZ80 is running in 24-bit ADL mode. For classic mode, 16 bit registers are passed as pointers to the MOS commands; these are automatically promoted to 24 bit by adding the MB register to bits 16-23 of the register. When running in ADL mode, a 24-bit register will be passed, but MB must be set to 0.

The following MOS commands are supported:

### `0x00`: mos_getkey

Read a keypress from the VDP

Parameters: None

Returns:

- `A`: The keycode of the character pressed

NB: This is a blocking function. This routine will wait and only return once a key is pressed.

### `0x01`: mos_load

Load a file from SD card

Parameters:

- `HL(U)`: Address of filename (zero terminated)
- `DE(U)`: Address at which to load
- `BC(U)`: Maximum allowed size (bytes)

Preserves: `HL(U)`, `DE(U)`, `BC(U)`

Returns:

- `A`: File error, or 0 if OK
- `F`: Carry reset if no room for file, otherwise set

### `0x02`: mos_save

Save a file to SD card

Parameters:

- `HL(U)`: Address of filename (zero terminated)
- `DE(U)`: Address to save from
- `BC(U)`: Number of bytes to save

Preserves: `HL(U)`, `DE(U)`, `BC(U)`

Returns:

- `A`: File error, or 0 if OK
- `F`: Carry set

### `0x03`: mos_cd

Change current directory on the SD card

Parameters:

- `HL(U)`: Address of path (zero terminated)

Preserves: `HL(U)`

Returns:

- `A`: File error, or 0 if OK

### `0x04`: mos_dir

List SD card folder contents to screen.

This is a simple directory listing command that will list the contents of the current directory to the screen.  More advanced directory listing functionality for applications to use is available via the [FatFS commands API](#fatfs-commands).

Parameters:

- `HL(U)`: Address of path (zero terminated)

Preserves: `HL(U)`

Returns:

- `A`: File error, or 0 if OK

### `0x05`: mos_del

Delete a file or folder from the SD card

Parameters:

- `HL(U)`: Address of path (zero terminated)

Preserves: `HL(U)`

Returns:

- `A`: File error, or 0 if OK

### `0x06`: mos_ren

Rename a file on the SD card

Parameters:

- `HL(U)`: Address of filename1 (zero terminated)
- `DE(U)`: Address of filename2 (zero terminated)

Preserves: `HL(U)`, `DE(U)`

Returns:

- `A`: File error, or 0 if OK

### `0x07`: mos_mkdir

Make a folder on the SD card

Parameters:

- `HL(U)`: Address of path (zero terminated)

Preserves: `HL(U)`

Returns:

- `A`: File error, or 0 if OK

### `0x08`: mos_sysvars

Fetch a pointer to the [system variables](#system-variables)

Parameters: None

Returns:

- `IXU`: Pointer to the MOS system variable area (this is always 24 bit)

### `0x09`: mos_editline

Invoke the line editor

Parameters:

- `HL(U)`: Address of the buffer
- `BC(U)`: Buffer length
- `E`: Flags to control editor behaviour

Preserves: `HL(U)`, `BC(U)`, `DE(U)`

Returns:

- `A`: Key that was used to exit the input loop (CR=13, ESC=27)

Editor behaviour flags are as follows:

| Bit | Description |
| --- | ----------- |
| 0   | When set, buffer will be cleared before use |
| 1   | When set, tab-completion for MOS commands and files is enabled * |
| 2   | When set, hotkeys are _disabled_ * |
| 3   | When set, input history will be _disabled_ * |
| 4-7 | Reserved for future use (for future compatibility, ensure these are set to zero) |

\* Support for editor control flags was added in Console8 MOS 2.2.0.  Prior to this the only documented values for `E` were 0 and 1 to indicate whether the buffer should be cleared.

### `0x0A`: mos_fopen

Get a file handle

Parameters:

- `HL(U)`: Address of filename (zero terminated)
- `C`: Mode

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: File handle, or 0 if couldn't open

Mode can be one of: fa_read, fa_write, fa_open_existing, fa_create_new, fa_create_always, fa_open_always or fa_open_append

NB: If you open the file using mos_fopen, you must close it using mos_fclose, not ffs_api_fclose

### `0x0B`: mos_fclose

Close a file handle

Parameters:

- `C`: File handle, or 0 to close all open files

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: Number of files still open

### `0x0C`: mos_fgetc

Get a character from an open file

Parameters:

- `C`: File handle

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: Character read
- `F`: C set if last character in file, otherwise NC (MOS 1.04 or greater)

### `0x0D`: mos_fputc

Write a character to an open file

Parameters:

- `C`: File handle
- `B`: Character to write

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

None

### `0x0E`: mos_feof

Check for end of file

Parameters:

- `C`: File handle

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: 1 if at end of file, otherwise 0

### `0x0F`: mos_getError

Copy an error string to a buffer

Parameters:

- `E`: The error code
- `HL(U)`: Address of buffer to copy message into
- `BC(U)`: Size of buffer

Preserves: `DE(U)`, `HL(U)`, `BC(U)`

Returns:

None

### `0x10`: mos_oscli

Execute a MOS command

Parameters:

- `HL(U)`: Pointer the the MOS command string

Preserves: `HL(U)`

Returns:

- `A`: MOS error code

NB previously documentation for this command was incorrect, as it documented additional parameters in `DE(U)` and `BC(U)`.  These registers are not currently used.

### `0x11`: mos_copy

Copy a file on the SD card

Parameters:

- `HL(U)`: Address of filename1 (zero terminated)
- `DE(U)`: Address of filename2 (zero terminated)

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: File error, or 0 if OK

NB: Requires MOS 1.03 or greater

### `0x12`: mos_getrtc

Get a time string from the RTC (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a buffer to copy the string to (at least 32 bytes)

Preserves: `HL(U)`

Returns:

- `A`: Length of time string

### `0x13`: mos_setrtc

Set the RTC (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a 6-byte buffer with the time data in

```
+0: Year (offset from 1980, so 1989 is 9)
+1: Month (1 to 12)
+2: Day of Month (1 to 31)
+3: Hour (0 to 23)
+4: Minute (0 to 59)
+5: Second (0 to 59)
```

Preserves: `HL(U)`

Returns:

None

### `0x14`: mos_setintvector

Set an interrupt vector (Requires MOS 1.03 or above)

Parameters:

- `E`: Interrupt vector number to set
- `HLU`: Address of new interrupt vector (24-bit pointer)

Preserves: `HLU`, `DEU`

Returns:

- `HL(U)`: Address of the previous interrupt vector (24-bit pointer)

### `0x15`: mos_uopen

Open UART1 (Requires MOS 1.03 or above)

To handle the received interrupts, you will need to assign a handler to UART1's interrupt vector (0x1A).

Parameters:

- `IXU`: Pointer to a UART struct

```
+0: Baud rate (24-bit, little endian)
+3: Data bits (5, 6, 7 or 8)
+4: Stop bits (1 or 2)
+5: Parity bits (0: None, 1: Odd, 3: Even)
+6: Flow control (0: None, 1: Hardware)
+7: Enabled interrupts
    - Bit 0: Set to enable received data interrupt
    - Bit 1: Set to enable transmit data interrupt
    - Bit 2: Set to enable line status change interrupt
    - Bit 3: Set to enable modem status change interrupt
    - Bit 4: Set to enable transmit complete interrupt
```

Preserves: `HL(U)`

Returns:

- `A`: Error code (always 0)

### `0x16`: mos_uclose

Close UART1 (Requires MOS 1.03 or above)

### `0x17`: mos_ugetc

Read a character from UART1 (Requires MOS 1.03 or above)

Returns:

- `A`: The character read
- `F`: C if successful, NC if the UART is closed

NB: If UART1 is open, this is a blocking function which means this routine will wait and only return once a character is received.

### `0x18`: mos_uputc

Write a character to UART1 (Requires MOS 1.03 or above)

Parameters:

- `C`: The character to write

Returns:

- `F`: C if successful, NC if the UART is closed

### `0x19`: mos_getfil

Get a pointer to a FIL structure in MOS (Requires MOS 1.03 or above)

Parameters:

- `C`: File handle

Preserves: `BC(U)`

Returns:

- `HLU`: 24-bit pointer to a FIL structure (in MOS RAM)

### `0x1A`: mos_fread

Read a block of data from a file (Requires MOS 1.03 or above)

Parameters:

- `C`: File handle
- `HLU`: Pointer to a buffer to read the data into
- `DEU`: Number of bytes to read

Preserves: `HL(U)`, `BC(U)`

Returns:

- `DEU`: Number of bytes read

### `0x1B`: mos_fwrite

Write a block of data to a file (Requires MOS 1.03 or above)

Parameters:

- `C`: File handle
- `HLU`: Pointer to a buffer that contains the data to write
- `DEU`: Number of bytes to write out

Preserves: `HL(U)`, `BC(U)`

Returns:

- `DEU`: Number of bytes written

### `0x1C`: mos_flseek

Move the read/write pointer in a file (Requires MOS 1.03 or above)

Parameters:

- `C`: File handle
- `HLU`: Least significant 3 bytes of the offset from the start of the file
- `E`: Most significant byte of the offset (set to 0 for files < 16MB)

Preserves: `HL(U)`, `BC(U)`, `DE(U)`

Returns:

- `A`: FRESULT

### `0x1D`: mos_setkbvector

Allows user programs to access VDP keyboard packets without overriding the entire uart0 interrupt handler.
The user program registered, will be called during the uart0 interrupt handler, being passed the address of the full VDP keyboard packet.

Parameters:

- `C`: Address length in HL (0 = 24bit, 1 = 16bit). If 1 then set the top byte of HLU(callback address) to MB (for ADL=0 callers)
- `HL(U)`: Callback address of user program to register, or 0 to clear any previously registered vector

Returns: Nothing upon registration. The user program can expect the full VDP packet address in DE(24-bit) upon entry.

Be sure to clear the kbvector before your program exits (call mos_setkbvector again with HL=0).

[example code](https://github.com/tomm/toms-agon-experiments/blob/main/custom_kbvector_demo/custom_kbvector_demo.asm) that registers a custom handler.

### `0x1E`: mos_getkbmap

Fetch a pointer to the virtual keyboard map (Requires MOS 1.04 RC2 or above)

Parameters: None

Returns:

- `IXU`: Pointer to the keyboard bitmap (this is always 24 bit)

### `0x1F`: mos_i2c_open

Open the I2C bus as Master (Requires MOS 1.04 RC3 or above)

Parameters:

- `C`: Frequency ID (1: 57600, 2: 115200, 3: 230400)

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns: None

### `0x20`: mos_i2c_close

Close the I2C bus (Requires MOS 1.04 RC3 or above)

Parameters: None

Preserves: `HL(U)`, `BC(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns: None

### `0x21`: mos_i2c_write

Write a block of bytes to the I2C bus (Requires MOS 1.04 RC3 or above)

Parameters:

- `C`: I2C Address
- `B`: Number of bytes to write (maximum 32)
- `HL(U)`: Pointer to a buffer to read the bytes from

Preserves: `HL(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: Status
	- `0`: OK
	- `1`: No response from I2C slave
	- `2`: Data NACK
	- `4`: Bus arbitration lost
	- `8`: Bus error

### `0x22`: mos_i2c_read

Read a block of bytes from the I2C bus (Requires MOS 1.04 RC3 or above)

Parameters:

- `C`: I2C Address
- `B`: Number of bytes to read (maximum 32)
- `HL(U)`: Pointer to a buffer to write the bytes to

Preserves: `HL(U)`, `DE(U)`, `IX(U)`, `IY(U)`

Returns:

- `A`: Status
	- `0`: OK
	- `1`: No response from I2C slave
	- `2`: Data NACK
	- `4`: Bus arbitration lost
	- `8`: Bus error

***

## FatFS commands

MOS makes use of FatFS to access the SD card.  Some of FatFS's functionality is exposed via the MOS API.  The exact API calls listed here may be expanded in later versions of MOS.

For more information on FatFS data structures and functions, see the [FatFS documentation](http://elm-chan.org/fsw/ff/00index_e.html).

### `0x80`: ffs_fopen

Open a file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to an empty FIL structure
- `DE(U)`: Pointer to a C (zero-terminated) filename string
- `C`: File open mode

Preserves: `HL(U)`, `DE(U)`, `C`

Returns:

- `A`: FRESULT

Example:

```
			LD	HL, fil				; FIL buffer
			LD	DE, filename			; Filename (0 terminated)
			LD	C, fa_read			; Mode
			MOSCALL	ffs_fopen			; Open the file
			LD	DE, buffer			; Where to store the read file
			LD	BC, 256				; Number of bytes to read
			MOSCALL	ffs_fread			; Read the data in
			PUSH	BC				; Preserve number of bytes read
			MOSCALL	ffs_fclose			; Close the file
			POP	BC				; BC: Number of bytes read
			RET

filename:		DB	"example.txt", 0		; The file to read

fil:			DS	FIL_SIZE			; FIL buffer (defined in mos_api.inc)
buffer:			DS	256				; Buffer for storing read data
```

### `0x81`: ffs_fclose

Close a file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FIL structure

Preserves: `HL(U)`

Returns:

- `A`: FRESULT

See ffs_fopen for an example

### `0x82`: ffs_fread

Read from a file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FIL structure
- `DE(U)`: Pointer to a buffer to store the data in
- `BC(U)`: Number of bytes to read (typically the size of the buffer)

Preserves: `HL(U)`, `DE(U)`

Returns:

- `BC(U)`: Number of bytes read
- `A`: FRESULT

See ffs_fopen for an example

### `0x83`: ffs_fwrite

Write to a file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FIL structure
- `DE(U)`: Pointer to a buffer to read the data from
- `BC(U)`: Number of bytes to write (typically the size of the buffer)

Preserves: `HL(U)`, `DE(U)`

Returns:

- `BC(U)`: Number of bytes written
- `A`: FRESULT

Example:

```
			LD	HL, fil				; FIL buffer
			LD	DE, filename			; Filename (0 terminated)
			LD	C, fa_write | fa_create_always	; Mode
			MOSCALL	ffs_fopen			; Open the file
			LD	DE, buffer			; Location of data to write
			LD	BC, 256				; Number of bytes to write
			MOSCALL	ffs_write			; Write the data
			MOSCALL	ffs_fclose			; Close the file
			RET

filename:	DB	"example.txt", 0		; The file to read

fil:		DS	FIL_SIZE			; FIL buffer (defined in mos_api.inc)
buffer:		DS	256				; Buffer containing data to write out
```

### `0x84`: ffs_flseek

Move the read/write pointer in a file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FIL structure
- `DE(U)`: Least significant 3 bytes of the offset from the start of the file
- `C`: Most significant byte of the offset (set to 0 for files < 16MB)

Preserves: `HL(U)`, `DE(U)`, `BC(U)`

### `0x85`: ffs_ftruncate

Truncate a file to the current file pointer offset (Requires Console8 MOS 2.3.0 or above)

To truncate to a specified size you will need to use ffs_flseek to move the file pointer to the desired location before calling ffs_ftruncate.

Parameters:

- `HL(U)`: Pointer to a FIL structure

Preserves: `HL(U)`


### `0x8E`: ffs_feof

Detect end of file (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FIL structure

Preserves: `HL(U)`

Returns:

- `A`: 1 if at the end of the file, otherwise 0

### `0x91`: ffs_dopen

Open a directory (Requires Console8 MOS 2.2.0 or above)

Parameters:

- `HL(U)`: Pointer to a blank DIR structure
- `DE(U)`: Pointer to a C (zero-terminated) directory path string

Preserves: `HL(U)`, `DE(U)`

Returns:

- `A`: FRESULT

### `0x92`: ffs_dclose

Close a directory (Requires Console8 MOS 2.2.0 or above)

Parameters:

- `HL(U)`: Pointer to a DIR structure

Preserves: `HL(U)`

Returns:

- `A`: FRESULT

### `0x93`: ffs_dread

Read next directory entry into a FILINFO data structure (Requires Console8 MOS 2.2.0 or above)

Parameters:

- `HL(U)`: Pointer to a DIR structure
- `DE(U)`: Pointer to a FILINFO structure

Preserves: `HL(U)`, `DE(U)`

Returns:

- `A`: FRESULT

### `0x96`: ffs_stat

Get file information (Requires MOS 1.03 or above)

Parameters:

- `HL(U)`: Pointer to a FILINFO structure
- `DE(U)`: Pointer to a C (zero-terminated) filename string

Preserves: `HL(U)`, `DE(U)`

Returns:

- `A`: FRESULT

Example:

```
			LD	HL, filinfo			; FILINFO buffer
			LD	DE, filename			; Filename (0 terminated)
			MOSCALL	ffs_stat
			RET

filename:	DB	"example.txt", 0		; The file to read

filinfo:	DS	FILINFO_SIZE			; FILINFO buffer (defined in mos_api.inc)
```

### `0x9E`: ffs_getcwd

Get the current working directory (Requires Console8 MOS 2.2.0 or above)

Parameters:

- `HL(U)`: Pointer to a buffer to store the directory path in
- `BC(U)`: Maximum length of the buffer

Preserves: `HL(U)`, `BC(U)`

Returns:

- `A`: FRESULT

***

## System Variables

The MOS API command [mos_sysvars](#0x08-mos_sysvars) returns a pointer to the base of the MOS system variables area in IXU as a 24-bit pointer. The MOS system variables are often simply referred to as sysvars.

The following system variables are available in [mos_api.inc](#usage-from-z80-assembler):

```
; System variable indexes for api_sysvars
; Index into _sysvars in globals.asm
;
sysvar_time:			EQU	00h	; 4: Clock timer in centiseconds (incremented by 2 every VBLANK)
sysvar_vpd_pflags:		EQU	04h	; 1: Flags to indicate completion of VDP commands
sysvar_keyascii:		EQU	05h	; 1: ASCII keycode, or 0 if no key is pressed
sysvar_keymods:			EQU	06h	; 1: Keycode modifiers
sysvar_cursorX:			EQU	07h	; 1: Cursor X position
sysvar_cursorY:			EQU	08h	; 1: Cursor Y position
sysvar_scrchar:			EQU	09h	; 1: Character read from screen
sysvar_scrpixel:		EQU	0Ah	; 3: Pixel data read from screen (R,B,G)
sysvar_audioChannel:	EQU	0Dh	; 1: Audio channel
sysvar_audioSuccess:	EQU	0Eh	; 1: Audio channel note queued (0 = no, 1 = yes)
sysvar_scrWidth:		EQU	0Fh	; 2: Screen width in pixels
sysvar_scrHeight:		EQU	11h	; 2: Screen height in pixels
sysvar_scrCols:			EQU	13h	; 1: Screen columns in characters
sysvar_scrRows:			EQU	14h	; 1: Screen rows in characters
sysvar_scrColours:		EQU	15h	; 1: Number of colours displayed
sysvar_scrpixelIndex:	EQU	16h	; 1: Index of pixel data read from screen
sysvar_vkeycode:		EQU	17h	; 1: Virtual key code from FabGL
sysvar_vkeydown			EQU	18h	; 1: Virtual key state from FabGL (0=up, 1=down)
sysvar_vkeycount:		EQU	19h	; 1: Incremented every time a key packet is received
sysvar_rtc:				EQU	1Ah	; 8: Real time clock data
sysvar_keydelay:		EQU	22h	; 2: Keyboard repeat delay
sysvar_keyrate:			EQU	24h	; 2: Keyboard repeat rate
sysvar_keyled:			EQU	26h	; 1: Keyboard LED status
sysvar_scrMode:			EQU	27h	; 1: Screen mode (from MOS 1.04)
sysvar_rtc_enable:		EQU 28h ; 1: RTC enable status (from Console8 MOS 2.0.0)
sysvar_mouseX:			EQU 29h ; 2: Mouse X position
sysvar_mouseY:			EQU 2Bh ; 2: Mouse Y position
sysvar_mouseButtons:	EQU 2Dh ; 1: Mouse left+right+middle buttons (bits 0-2, 0=up, 1=down)
sysvar_mouseWheel:		EQU 2Eh ; 1: Mouse wheel delta
sysvar_mouseXDelta:		EQU 2Fh ; 2: Mouse X delta
sysvar_mouseYDelta:		EQU 31h ; 2: Mouse Y delta
```
Example: Reading a virtual keycode in ADL mode (24-bit):
```
		MOSCALL	mos_getkey
		LD	A, (IX + sysvar_vkeycode)	; Load A with the virtual keycode from FabGL
```
Example: Reading a virtual keycode in Z80 mode (16-bit):
```
		MOSCALL	mos_getkey
		LD.LIL	A, (IX + sysvar_vkeycode)	; Load A with the virtual keycode from FabGL
```
