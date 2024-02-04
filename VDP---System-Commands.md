# VDP System Commands

`VDU 23, 0` is reserved for commands sent to the VDP.  These are largely unique to the Agon, and are used to access some of the more sophisticated features of the VDP.

Some of the commands are used for operating system functionality, and will not usually need to be used by user applications, whilst others allow access to features that applications will want to use.

Please note that not all versions of the VDP support the complete command set.  The following list uses the following symbols to indicate which VDP versions support which commands:

 \* Requires VDP 1.03 or above<br>
 \** Requires VDP 1.04 or above<br>
 § Requires Console8 VDP 2.3.0 or above<br>
 §§ Requires Console8 VDP 2.4.0 or above

Commands between &80 and &89 will return their data back to the eZ80 via the [serial protocol](#serial-protocol).

NB:

- Prior to MOS 1.03 the subset commands that it supported were indexed from &00, not &80. For example, `VDU 23, 0, &02` to request the cursor position.


## `VDU 23, 0, &80, n`: General poll

This command will echo back `n` to MOS (see [Serial Protocol](#serial-protocol))

This command is used by MOS and the VDP to synchronise with each other during the system start-up process.  It is not intended to be used by applications.

## `VDU 23, 0, &81, n`: Set the keyboard locale

Sets the keyboard to a given locale/format.  The following locales are supported:

| Locale | Description |
| ------ | ----------- |
| 0 | UK |
| 1 | US |
| 2 | German |
| 3 | Italian |
| 4 | Spanish |
| 5 | French |
| 6 | Belgian |
| 7 | Norwegian |
| 8 | Japanese ** |
| 9 | US International ** |
| 10 | US International Alternate ** |
| 11 | Swiss German ** |
| 12 | Swiss French ** |
| 13 | Danish ** |
| 14 | Swedish ** |
| 15 | Portuguese ** |
| 16 | Brazilian Portuguese § |
| 17 | Dvorak § |

Any other value will be interpreted as UK.

## `VDU 23, 0, &82`: Request text cursor position

This command will return the current text cursor position to MOS.  Once the cursor position has been returned the text cursor position data inside MOS's system variables will be updated to reflect the current cursor position.

## `VDU 23, 0, &83, x; y;`: Get ASCII code of character at character position x, y

This command will return the ASCII code of the character at the given character position to MOS.

This command works by comparing pixels on the screen at the given position to the currently defined character set, using the currently selected text colour.  This means that it may not always be accurate, or able to succeed.  Redefining characters after they have been drawn, or changing the text colour, may cause this command to fail.

This command will not recognise characters that have been mapped to bitmaps using `VDU 23, 0, &92, char, bitmapId;`.

## `VDU 23, 0, &84, x; y;`: Get colour of pixel at pixel position x, y

This command will return the colour of the pixel at the given pixel position to MOS.  The corresponding MOS system variables will be updated to reflect the read pixel colour.

## `VDU 23, 0, &85, channel, command, <args>`: Audio commands

Sends a command to the [VDP Enhanced Audio API](VDP---Enhanced-Audio-API.md) **

Prior to VDP 1.04 this command could only perform what is now audio command zero, which plays a note on a channel.

## `VDU 23, 0, &86`: Fetch the screen dimensions

Returns the screen dimensions to MOS.  Generally applications should not need to call this, as this information will be automatically sent to MOS when the screen mode is changed.

## `VDU 23, 0, &87`: RTC control *

This command controls the Real Time Clock within the Agon VDP.

- `VDU 23, 0, &87, 0`: Read the RTC
  - a data packet will be sent to MOS with the current RTC data, and MOS system variables updated accordingly

- `VDU 23, 0, &87, 1, y, m, d, h, m, s`: Set the RTC


## `VDU 23, 0, &88, delay; rate; led`: Keyboard Control *

This command controls the keyboard repeat delay, repeat rate, and LED status.

Only delay values between 250 and 1000 are supported, and represents a time in milliseconds before the key will start repeating.

The rate value represents the time between key repeats, and is given in milliseconds.  Values from 33-500 are supported.

The LED value is a bit mask that controls the state of the keyboard LEDs.  The following bits are defined:

| Bit | Name | Meaning |
| --- | ---- | ------- |
| 0 | Scroll Lock | If set then the Scroll Lock LED will be turned on.  If clear then it will be turned off. |
| 1 | Caps Lock | If set then the Caps Lock LED will be turned on.  If clear then it will be turned off. |
| 2 | Num Lock | If set then the Num Lock LED will be turned on.  If clear then it will be turned off. |

## `VDU 23, 0, &89, command, [<args>]`: Mouse control **

Commands beginning with `VDU 23, 0, &89` are reserved for mouse control, and are implemented from VDP 1.04 onwards.

### `VDU 23, 0, &89, 0`: Enable the mouse

Enables the mouse cursor, and will start sending mouse data packets to MOS.

If there is no mouse connected then this command will have no effect.

### `VDU 23, 0, &89, 1`: Disable the mouse

Disables the mouse cursor, and will stop sending mouse data packets to MOS.

### `VDU 23, 0, &89, 2`: Reset the mouse

Resets the mouse system restoring default settings.

### `VDU 23, 0, &89, 3, cursorId;`: Set mouse cursor

Sets the mouse cursor to the given cursor ID.

There are several built-in mouse cursors that are available for use.  These have been inherited from fab-gl and are numbered from 0-18.  The "Cursor" column in the table below shows the fab-gl name for the cursor.

| ID | Cursor | Description |
| -- | ------ | ----------- |
| 0  | PointerAmigaLike | 11x11 Amiga like colored mouse pointer |
| 1  | PointerSimpleReduced | 10x15 mouse pointer |
| 2  | PointerSimple | 11x19 mouse pointer |
| 3  | PointerShadowed | 11x19 shadowed mouse pointer |
| 4  | Pointer | 12x17 mouse pointer |
| 5  | Pen | 16x16 pen |
| 6  | Cross1 | 9x9 cross |
| 7  | Cross2 | 11x11 cross |
| 8  | Point | 5x5 point |
| 9  | LeftArrow | 11x11 left arrow |
| 10 | RightArrow | 11x11 right arrow |
| 11 | DownArrow | 11x11 down arrow |
| 12 | UpArrow | 11x11 up arrow |
| 13 | Move | 19x19 move |
| 14 | Resize1 | 12x12 resize orientation 1 |
| 15 | Resize2 | 12x12 resize orientation 2 |
| 16 | Resize3 | 11x17 resize orientation 3 |
| 17 | Resize4 | 17x11 resize orientation 4 |
| 18 | TextInput | 7x15 text input |

Additional cursors can be defined using the `VDU 23, 27, &40, hotX, hotY` command.  For details of that command see the [Bitmaps API](VDP---Bitmaps-API.md) documentation.  Using that API it is possible to define a custom mouse cursor using a bitmap, which can then be selected using this command passing in the 16-bit bitmapId in place of a cursorId.

### `VDU 23, 0, &89, 4, x; y;`: Set mouse cursor position

Explicitly moves the mouse cursor to a given position on the screen.

### `VDU 23, 0, &89, 5, x1; y1; x2; y2;`: Reserved

This command is reserved for future use.  It is not yet implemented.

(When implemented it will set the mouse area, which will restrict the mouse cursor to a given area of the screen.  The mouse cursor will not be able to leave this area.)

### `VDU 23, 0, &89, 6, sampleRate`: Set mouse sample rate

Sets the rate at which mouse data will be sampled.  The rate given is a number of samples per second.  The default rate is 60.

Valid sample rates are 10, 20, 40, 60, 80, 100, and 200 (samples/sec).  Any other values will be ignored.

### `VDU 23, 0, &89, 7, resolution`: Set mouse resolution

Sets the mouse resolution.  Values in the range 0-3 are supported, or 255 (-1) to pick the default resolution.  The default resolution is 2.

The following resolutions are supported:

| Value | Resolution |
| ----- | ---------- |
| 0 | 1 count per mm (25 dpi) |
| 1 | 2 counts per mm (50 dpi) |
| 2 | 4 counts per mm (100 dpi) (default) |
| 3 | 8 counts per mm (200 dpi) |

### `VDU 23, 0, &89, 8, scaling`: Set mouse scaling

Sets the mouse scaling factor.  Only values `1` and `2` are supported to indicate 1:1 or 1:2 scaling.  Additionally a value of `0` can be sent to indicate "default" scaling, which is 1:1.

### `VDU 23, 0, &89, 9, acceleration;`: Set mouse acceleration

This sets the mouse acceleration factor to a 16-bit value.  Setting the value to `0` will result in the default acceleration being used, which is 180.  The suggested range for this is 0-2000.

### `VDU 23, 0, &89, 10, wheelAcceleration; wheelAccHighByte`: Set mouse wheel acceleration (accepts a 24-bit value)

Sets the wheel acceleration factor to a 24-bit value.  Setting the value to `0` will result in the default acceleration being used, which is 60000.  The suggested range for this is 0-100000.

### Mouse data packets

Mouse data packets are sent in response to all of the above commands, and if the mouse has been enabled whenever the mouse is moved.  This ensures that mouse data is constantly updated in MOS system variables.


## `VDU 23, 0, &90, n, b1, b2, b3, b4, b5, b6, b7, b8`: Redefine character n (0-255) with 8 bytes of data §

This command works identically to `VDU 23, n, b1, b2, b3, b4, b5, b6, b7, b8`, but allows characters 0-31 to also be redefined.

## `VDU 23, 0, &91`: Reset all characters to original definition §

This command will reset all characters to their original definitions.

## `VDU 23, 0, &92, char, bitmapId;`: Map character char to display bitmapId §§

This command will map a character to a bitmap.  The bitmap ID given must be a valid bitmap ID.  The purpose of this command is to allow for fast and efficient drawing of bitmaps to the screen, by allowing them to be drawn as characters.

Any character number can be used, including those in the range of 0-31, or the usual/normal alphabetic range.  This could be used, for instance, to create a custom colour font.  It could alternatively be used with characters in the range of 128-255 to define a custom graphical tile-set, like PETSCII but in colour.

When a character has been mapped to use a bitmap the bitmap will be used in place of that character, and will be drawn at the current text cursor position, placing the bottom left of the bitmap at the cursor position.  The cursor position will then be moved to the right by the width of a character, and _not_ by the width of the bitmap.

That last point is important.  It means that if you use a bitmap that is larger than a character, then the next character will overlap the bitmap.  Similarly bitmaps that are taller than a character will overwrite the line of text above the current cursor.  This can be used to create some interesting effects, but can also be a source of visual bugs if you are not careful.  If you are using oversized bitmaps, then using the `VDU 9` to move forward an additional character may be useful.


## `VDU 23, 0, &94, n`: Read colour palette entry n (returns a pixel colour data packet) §§

This command will return the colour of the given palette entry to MOS.  This data is sent using a "screen pixel" data packet.  The corresponding MOS system variables related to screen pixel colour will be updated to reflect the read palette entry.

The Agon VDP system supports a 64 colour palette, so values in the range of 0-63 will return data on that palette entry.  The following special values are also supported:

| Value | Description |
| ----- | ----------- |
| 128 | The current text foreground colour |
| 129 | The current text background colour |
| 130 | The current graphics foreground colour |
| 131 | The current graphics background colour |

Any other colour value will not be recognise, and no response sent.

## `VDU 23, 0, &A0, bufferId, command, <args>`: Buffered command API **

Send a command to the [VDP Buffered Commands API](VDP---Buffered-Commands-API.md)

## `VDU 23, 0, &A1`: Update VDP (for exclusive use of the agon-flash tool) **

This command is used by the agon-flash tool to update the VDP firmware.  It should not be used by any other software.

## `VDU 23, 0, &C0, n`: Turn logical screen scaling on and off *

Turns logical screen scaling on and off, where 1=on and 0=off.

When logical scaling is turned off, the graphics system will no longer use the 1280x1024 logical coordinate system and instead use pixel coordinates.  The screen origin point at 0,0 will change to be the top left of the screen, and the Y axis will go down the screen instead of up.  

For more information, see the [Screen modes documentation](VDP---Screen-Modes.md).

## `VDU 23, 0, &C1, n`: Switch legacy modes on or off **

Turns legacy screen modes on and off, where 1=on and 0=off.

By default, the original screen modes 0-4 are not available and are instead replaced by new modes that are more compatible with modern monitors.  For compatibility with older software, written for Agon systems running earlier versions of the VDP firmware, this command can be used to switch back to those original, legacy, screen modes.

For more information, see the [Screen modes documentation](VDP---Screen-Modes.md).


## `VDU 23, 0, &C3`: Swap the screen buffer and/or wait for VSYNC **

Swap the screen buffer (double-buffered modes only) or wait for VSYNC (all modes).

This command will swap the screen buffer, if the current screen mode is double-buffered, doing so at the next VSYNC.  If the current screen mode is not double-buffered then this command will wait for the next VSYNC signal before returning.  This can be used to synchronise the screen with the vertical refresh rate of the monitor.

Waiting for VSYNC can be useful for ensuring smooth graphical animation, as it will prevent tearing of the screen.

(In BASIC performing a `*FX 19` command will perform a similar wait for VSYNC, but on the eZ80 side of the system, but will not swap the screen buffer.)


## `VDU 23, 0, &FE, n`: Console mode **

Turns "console mode" on and off, where 1=on and 0=off (the default).

This mode is primarily intended for use with debugging tooling.  It will reflect VDU command bytes through to a serial terminal attached to the VDPs USB port, and will attempt to pass keyboard input from the serial terminal back through the VDP to MOS.

This mode is not intended for general use.  If you are looking for a way to output data to the attached serial terminal then `VDU 2` is a better option.


## `VDU 23, 0, &FF`: Switch to or resume "terminal mode"

This command enables "terminal mode", which changges the behaviour of the VDP to be more like a traditional terminal.  This is useful for running CP/M in place of MOS on the eZ80, or other software that expects to be running on a terminal.

When terminal mode is enabled, VDU commands will no longer be recognised and processed, keyboard entry in BBC BASIC/MOS is no longer supported, and the VDP protocol is essentially suspended.  Instead the VDP acts as a dumb terminal, and will send all keyboard input to the eZ80's UART0.  The eZ80 can then process this input as it sees fit.

By default a VT100 terminal is emulated, but this can be changed by sending the appropriate escape sequences to the VDP.  Terminal support is inherited from FabGL, and so its [custom escape sequences](http://www.fabglib.org/special_term_escapes.html) can be used to change the terminal behaviour.

From Console8 VDP version 2.2.0, the following additional escape sequences are supported:

| Sequence | Description |
| -------- | ----------- |
| `ESC + "_#Q!$"` | Quit/end terminal mode |
| `ESC + "_#S!$"` | Suspend terminal mode |

Quitting terminal mode returns the VDP back to normal operation, and will resume the VDP protocol.  The screen mode will be reset to the mode that was active before terminal mode was enabled.

Suspending terminal mode temporarily restores VDU command processing.  (Keyboard handling is unchanged.)  This could be useful to use VDU commands to perform drawing operations.  To resume terminal mode, `VDU 23, 0, &FF` must be sent again.



## Serial Protocol

Data sent from the VDP to the eZ80's UART0 is sent as a packet in the following format:

- cmd: The packet command, with bit 7 set
- len: Number of data bytes
- data: The data byte(s)

Words are 16 bit, and sent in little-endian format

In general, as a programmer using an Agon you should not need to worry about the format and contents of any of these packets, as they are handled by MOS.  On receipt of one of these packets, MOS will set system variables accordingly.  It also sets a bit in the VDPProtocol status byte corresponding to the type of data packet received.

If you are performing a command where you need to wait for a response from the VDP, then you will need to wait for the appropriate bit to be set in the VDP protocol byte.  Before you send a command to the VDP you should clear the bit that relates to the command you are about to send.  Once the command has been processed the VDP will set the bit again.  If the bit is not set then there may have been an error processing the command.

Packets:

- `0x00, value`: General Poll
- `0x01, keycode, modifiers, vkey, keydown`: Keyboard state
- `0x02, x, y`: Cursor position
- `0x03, char`: Character read from screen
- `0x04, r, g, b, index`: Pixel colour read from screen
- `0x05, channel, status`: Audio command status (see [VDP Enhanced Audio API](VDP---Enhanced-Audio-API.md))
- `0x06, width; height; cols, rows, colours`: Screen dimensions - width and height are words
- `0x07, year, month, day, dayOfYear, dayOfWeek, hour, minute, second`: RTC data *
- `0x08, delay, rate, led`: Keyboard status - delay and rate are words
- `0x09, x; y; buttons, wheelDelta, deltaX; deltaY;`: Mouse status - x, y, deltaX and deltaY are words

\* as of VDP 1.04 the RTC data is sent in a packed format.  This is interpreted by MOS and expanded into the appropriate system variables.  Prior to VDP 1.04 the `dayOfYear` value could be incorrect, as it cannot be guaranteed to fit into a byte.

MOS VDP Protocol flag bits:

| Bit | Description |
| --- | ----------- |
| 0 | Cursor position |
| 1 | Character read from screen |
| 2 | Point data (pixel colour) read from screen |
| 3 | Audio status |
| 4 | Mode info / Screen dimensions |
| 5 | RTC data |
| 6 | Mouse status |


### Keyboard

When a key is pressed, a packet is sent with the following data:
- cmd: 0x01
- keycode: The ASCII value of the key pressed
- modifiers: A byte with the following bits set (1 = pressed):
```
0. CTRL
1. SHIFT
2. ALT LEFT
3. ALT RIGHT
4. CAPS LOCK
5. NUM LOCK
6. SCROLL LOCK
7. GUI
```
From VDP 1.03, the following key data is also returned
- vkey: The FabGL virtual keycode
- keydown: 1 if the key is down, 0 if the key is up


