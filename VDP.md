# What is the VDP

The VDP is the Agon's Visual Display Processor. It is responsible for:

* Video output via the VGA connector
* Audio output via the built-in buzzer and audio jack
* Keyboard input via a PS/2 connector
* Mouse input via a PS/2 connector (on the Agon Console8, or with an adapter on the Agon Light)

It runs on the ESP32-Pico-D4 co-processor and uses a fork of the FabGL library (known as vdp-gl) to support those functions.

At a higher level, its input is a byte stream from the eZ80F92 main CPU over an internal high-speed UART connection @ 1,152,000 baud (384,000 baud for versions of MOS/VDP prior to 1.03). This stream contains a mixture of text and control characters. These control characters are mapped to the BBC BASIC VDU control characters, a choice made as BBC BASIC for Agon is the pre-installed programming language of Agon.

The ESP32 also outputs data back to the eZ80F92, for example keyboard data and screen information via a custom serial protocol.

## Executing a VDU sequence

### From BBC BASIC

The VDU command interprets each of the integer values following the keyword as an ASCII value. Values between 0 and 255 are accepted, and are separated by commas. If the value is immediately followed by a semicolon `;` then the value is a little-endian word from 0 to 65535.

Example:

`VDU 25, 69, 640; 512;`: Plot a dot in the center of the screen

### From MOS (Version 1.03 or greater)

The VDU command accepts values between 0 and 255 and are separated by spaces.

Example:

`VDU 17 15`: Set the text foreground colour to 15

## VDU Character Sequences

The aim is that the Agon's VDP should be as compatible as practical with the BBC Micro's VDU command, as well as the VDU commands supported by later versions of Acorn and R.T.Russell's BBC BASICs.  Where necessary, some extensions have been added to help facilitate the Agon's unique features and architecture.

The following VDU sequences are supported:

- `VDU 0`: Null (no operation)
- `VDU 4`: Write text at text cursor
- `VDU 5`: Write text at graphics cursor
- `VDU 7`: Make a short beep (BEL)
- `VDU 8`: Move cursor back one character
- `VDU 9`: Move cursor forward one character
- `VDU 10`: Move cursor down one line
- `VDU 11`: Move cursor up one line
- `VDU 12`: Clear text area (`CLS`)
- `VDU 13`: Carriage return
- `VDU 14`: Page mode ON (VDP 1.03 or greater)
- `VDU 15`: Page mode OFF (VDP 1.03 or greater)
- `VDU 16`: Clear graphics area (`CLG`)
- `VDU 17, colour`: Define text colour (`COLOUR`)
- `VDU 18, mode, colour`: Define graphics colour (`GCOL mode, colour`)
- `VDU 19, l, p, r, g, b`: Define logical colour (`COLOUR l, p` / `COLOUR l, r, g, b`)
- `VDU 22, n`: Select screen mode (`MODE n`)
- `VDU 23, n`: Re-program display character / System Commands
- `VDU 24, left; bottom; right; top;`: Set graphics viewport (VDP 1.04 or greater)
- `VDU 25, mode, x; y;`: PLOT mode, x, y
- `VDU 26`: Reset graphics and text viewports (VDP 1.04 or greater)
- `VDU 27, char`: Output character to screen (Agon Console8 VDP 2.3.0 or later)
- `VDU 28, left, bottom, right, top`: Set text viewport (VDP 1.04 or greater)
- `VDU 29, x; y;`: Graphics origin
- `VDU 30`: Home cursor
- `VDU 31, x, y`: TAB(x, y)
- `VDU 127`: Backspace

All other characters, i.e. those in the range of 32 to 126 and 128 to 255, are sent to the screen as ASCII, unaltered.

Any VDU command that is not recognised (such as `VDU 1`) will be ignored.


## VDU 23, 0: VDP commands

VDU 23, 0 is reserved for commands sent to the VDP

- `VDU 23, 0, &80, n`: General poll, which echoes back `n` to MOS (see [Serial Protocol](#serial-protocol))
- `VDU 23, 0, &81, n`: Set the keyboard locale (0=UK, 1=US, etc) 
- `VDU 23, 0, &82`: Request text cursor position
- `VDU 23, 0, &83, x; y;`: Get ASCII code of character at character position x, y
- `VDU 23, 0, &84, x; y;`: Get colour of pixel at pixel position x, y
- `VDU 23, 0, &85, channel, command, <args>`: Send a command to the [VDP Enhanced Audio API](VDP---Enhanced-Audio-API.md) **
- `VDU 23, 0, &86`: Fetch the screen dimensions
- `VDU 23, 0, &87`: RTC control *
- `VDU 23, 0, &88, delay; rate; led`: Keyboard Control *
- `VDU 23, 0, &89, command, [<args>]`: Mouse control **
- `VDU 23, 0, &90, n, b1, b2, b3, b4, b5, b6, b7, b8`: Redefine character n (0-255) with 8 bytes of data §
- `VDU 23, 0, &91`: Reset all characters to original definition §
- `VDU 23, 0, &92, char, bitmapId;`: Map character char to display bitmapId §§
- `VDU 23, 0, &94, n`: Read colour palette entry n (returns a pixel colour data packet) §§
- `VDU 23, 0, &A0, bufferId, command, <args>`: Send a command to the [VDP Buffered Commands API](VDP---Buffered-Commands-API.md) **
- `VDU 23, 0, &A1`: Update VDP (for exclusive use of the agon-flash tool) **
- `VDU 23, 0, &C0, n`: Turn logical screen scaling on and off, where 1=on and 0=off *
- `VDU 23, 0, &C1, n`: Switch legacy modes on or off **
- `VDU 23, 0, &C3`: Flip the screen buffer (double-buffered modes only) or wait for VSYNC (all modes) **
- `VDU 23, 0, &FF`: Switch to or resume terminal mode for CP/M (This will disable keyboard entry in BBC BASIC/MOS)

 \* Requires VDP 1.03 or above<br>
 \** Requires VDP 1.04 or above<br>
 § Requires Console8 VDP 2.3.0 or above<br>
 §§ Requires Console8 VDP 2.4.0 or above
 

Commands between &82 and &89 will return their data back to the eZ80 via the [serial protocol](#serial-protocol).

NB:

- Prior to MOS 1.03 these were indexed from &00, not &80. For example, `VDU 23, 0, &02` to request the cursor position.

## RTC control

- `VDU 23, 0, 7, 0`: Read the RTC
- `VDU 23, 0, 7, 1, y, m, d, h, m, s`: Set the RTC

## Mouse control

Commands beginning with `VDU 23, 0, &89` are reserved for mouse control, and are implemented from VDP 1.04 onwards.

- `VDU 23, 0, &89, 0`: Enable the mouse
- `VDU 23, 0, &89, 1`: Disable the mouse
- `VDU 23, 0, &89, 2`: Reset the mouse
- `VDU 23, 0, &89, 3, cursorId;`: Set mouse cursor
- `VDU 23, 0, &89, 4, x; y;`: Set mouse cursor position
- `VDU 23, 0, &89, 5, x1; y1; x2; y2;`: Reserved (Set mouse area - not yet implemented)
- `VDU 23, 0, &89, 6, sampleRate;`: Set mouse sample rate
- `VDU 23, 0, &89, 7, resolution;`: Set mouse resolution
- `VDU 23, 0, &89, 8, scaling`: Set mouse scaling
- `VDU 23, 0, &89, 9, acceleration;`: Set mouse acceleration
- `VDU 23, 0, &89, 10, wheelAcceleration; wheelAccHighByte`: Set mouse wheel acceleration (accepts a 24-bit value)

## VDU 23, 1: Cursor display

- `VDU 23, 1, 0`: Disable the text cursor
- `VDU 23, 1, 1`: Enable the text cursor

## VDU 23, 7: Scrolling

- `VDU 23, 7, extent, direction, speed`: Scroll the screen

## VDU 23, 16: Define Cursor Behaviour (Requires VDP 1.04 or greater)

- `VDU 23, 16, setting, mask`: Specify [cursor behaviour](#cursor-behaviour) by ANDing with mask then XORing with setting

## VDU 23, 27: Bitmaps, sprites, and mouse cursor

VDU 23, 27 is reserved for the bitmap, sprite, and mouse cursor functionality

### Bitmaps

- `VDU 23, 27, 0, n`: Select bitmap n (equating to buffer ID numbered 64000+`n`)
- `VDU 23, 27, 1, w; h; b1, b2 ... bn`: Load colour bitmap data into current bitmap
- `VDU 23, 27, 2, w; h; col1; col2;`: Create a solid colour rectangular bitmap (col1 and col2 form a 32 bit number in RGBA colour range.)
- `VDU 23, 27, 3, x; y;`: Draw current bitmap on screen at pixel position x, y (a valid bitmap must be selected first)
- `VDU 23, 27, &20, bufferId;`: Select bitmap using a 16-bit buffer ID
- `VDU 23, 27, &21, w; h; format`: Create bitmap from selected buffer

### Sprites

- `VDU 23, 27, 4, n`: Select sprite n
- `VDU 23, 27, 5`: Clear frames in current sprite
- `VDU 23, 27, 6, n`: Add bitmap n as a frame to current sprite (where bitmap's buffer ID is 64000+`n`)
- `VDU 23, 27, 7, n`: Activate n sprites
- `VDU 23, 27, 8`: Select next frame of current sprite
- `VDU 23, 27, 9`: Select previous frame of current sprite
- `VDU 23, 27, 10, n`: Select the nth frame of current sprite
- `VDU 23, 27, 11`: Show current sprite
- `VDU 23, 27, 12`: Hide current sprite
- `VDU 23, 27, 13, x; y;`: Move current sprite to pixel position x, y
- `VDU 23, 27, 14, x; y;`: Move current sprite by x, y pixels
- `VDU 23, 27, 15`: Update the sprites in the GPU
- `VDU 23, 27, 16`: Reset bitmaps and sprites and clear all data
- `VDU 23, 27, 17`: Reset sprites (only) and clear all data
- `VDU 23, 27, &26, n;`: Add bitmap n as a frame to current sprite using a 16-bit buffer ID

### Mouse cursor

- `VDU 23, 27, &40, hotX, hotY`: Setup a mouse cursor with a hotspot at hotX, hotY from the currently selected bitmap

## Serial Protocol

Data sent from the VDP to the eZ80's UART0 is sent as a packet in the following format:

- cmd: The packet command, with bit 7 set
- len: Number of data bytes
- data: The data byte(s)

Words are 16 bit, and sent in little-endian format

In general, as a programmer using an Agon you should not need to worry about any of these packets, as they are handled by MOS.  On receipt of one of these packets MOS sets system variables accordingly.  It will set a bit in the VDPProtocol status byte, and then whichever other system variables are relevant to the packet received.


Packets:

- `0x00`: General Poll
- `0x01, keycode, modifiers, vkey, keydown`: Keyboard state
- `0x02, x, y`: Cursor position
- `0x03, char`: Character read from screen
- `0x04, r, g, b, index`: Pixel colour read from screen
- `0x05, channel, status`: Audio command status (see [VDP Enhanced Audio API](VDP---Enhanced-Audio-API.md))
- `0x06, width; height; cols, rows, colours`: Screen dimensions - width and height are words
- `0x07, year, month, day, dayOfYear, dayOfWeek, hour, minute, second`: RTC data
- `0x08, delay, rate, led`: Keyboard status - delay and rate are words
- `0x09, x; y; buttons, wheelDelta, deltaX; deltaY;`: Mouse status - x, y, deltaX and deltaY are words

## Keyboard

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

## Cursor Behaviour

The following bits are implemented in VDU 23, 16

- `Bit 0 = 1`: Enable scroll protection - text cursor will not scroll when it moves off the bottom/right of the viewport
- `Bit 0 = 0`: Disable scroll protection (default)
- `Bit 4 = 1`: Text cursor will wrap to top of screen when it moves off the bottom of the screen
- `Bit 4 = 0`: Text cursor will scroll when it moves off the bottom of the screen (default)
- `Bit 5 = 1`: Cursor does not move right after a character is printed
- `Bit 5 = 0`: Cursor moves right after a character is printed (default)
- `Bit 6 = 1`: Graphics cursor (VDU 5 mode) carries on off edge of graphics viewport
- `Bit 6 = 0`: Graphics cursor does an implicit cr/lf when it moves off right of graphics viewport (default)

## Screen Modes

Modes over 128 are double-buffered

### Prior to Version 1.04

| Mode | Horz | Vert | Cols | Refresh |
|-----:|-----:|-----:|-----:|--------:|
| 0    | 1024 |  768 |    2 | 60hz    |
| 1    |  512 |  384 |   16 | 60hz    |
| 2    |  320 |  200 |   64 | 75hz    |
| 3    |  640 |  480 |   16 | 60hz    |

### From Version 1.04 or greater

| Mode | Horz | Vert | Cols | Refresh |
|-----:|-----:|-----:|-----:|--------:|
|    0 |  640 |  480 |   16 |    60hz |
|    1 |  640 |  480 |    4 |    60hz |
|    2 |  640 |  480 |    2 |    60hz |
|    3 |  640 |  240 |   64 |    60hz |
|    4 |  640 |  240 |   16 |    60hz |
|    5 |  640 |  240 |    4 |    60hz |
|    6 |  640 |  240 |    2 |    60hz |
|    8 |  320 |  240 |   64 |    60hz |
|    9 |  320 |  240 |   16 |    60hz |
|   10 |  320 |  240 |    4 |    60hz |
|   11 |  320 |  240 |    2 |    60hz |
|   12 |  320 |  200 |   64 |    70hz |
|   13 |  320 |  200 |   16 |    70hz |
|   14 |  320 |  200 |    4 |    70hz |
|   15 |  320 |  200 |    2 |    70hz |
|   16 |  800 |  600 |    4 |    60hz |
|   17 |  800 |  600 |    2 |    60hz |
|   18 | 1024 |  768 |    2 |    60hz |
|  129 |  640 |  480 |    4 |    60hz |
|  130 |  640 |  480 |    2 |    60hz |
|  132 |  640 |  240 |   16 |    60hz |
|  133 |  640 |  240 |    4 |    60hz |
|  134 |  640 |  240 |    2 |    60hz |
|  136 |  320 |  240 |   64 |    60hz |
|  137 |  320 |  240 |   16 |    60hz |
|  138 |  320 |  240 |    4 |    60hz |
|  139 |  320 |  240 |    2 |    60hz |
|  140 |  320 |  200 |   64 |    70hz |
|  141 |  320 |  200 |   16 |    70hz |
|  142 |  320 |  200 |    4 |    70hz |
|  143 |  320 |  200 |    2 |    70hz |

