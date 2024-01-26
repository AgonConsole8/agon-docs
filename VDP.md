# What is the VDP

The VDP is the Agon's Visual Display Processor. It is responsible for:

* Video output via the VGA connector
* Audio output via the built-in buzzer and audio jack
* Keyboard input via a PS/2 connector
* Mouse input via a PS/2 connector (on the Agon Console8, or with an adapter on the Agon Light)

It runs on the ESP32-Pico-D4 co-processor and uses a fork of the FabGL library (known as vdp-gl) to support those functions.

At a higher level, its input is a byte stream from the eZ80F92 main CPU over an internal high-speed UART connection @ 1,152,000 baud (384,000 baud for versions of MOS/VDP prior to 1.03). This stream contains a mixture of text and control characters. These control characters are mapped to the BBC BASIC VDU control characters, a choice made as BBC BASIC for Agon is the pre-installed programming language of Agon.  As a result, we refer to the commands that the VDP understands as VDU commands.

The ESP32 also outputs data back to the eZ80F92, for example keyboard data and screen information via a custom serial protocol.

## Executing a VDU sequence

### From BBC BASIC

The `VDU` statement in BBC BASIC essentially just means "send this data to the VDP".  It will accept any number of integer arguments between 0 and 255, separated by commas.  If a value is immediately followed by a semicolon `;` instead of a comma then the value is a little-endian word from 0 to 65535.

Example:

`VDU 25, 69, 640; 512;`: Plot a dot in the center of the screen

`VDU 65`: Print the letter "A", without a newline

A single `VDU` statement in BASIC can potentially contain multiple commands for the VDP to interpret, or a VDU command could instead be split across multiple `VDU` statements.  Other BASIC keywords that generate screen output are effectively just wrappers for VDU command sequences.

For example, the following code snippets are all directly equivalent, and result in an identical command stream being sent to the VDP:

```
PLOT 69, 640, 512
PRINT "A";
```
```
VDU 25, 69, 640; 512; 65
```
```
VDU 25
VDU 69
VDU 640; 512;
VDU ASC("A")
```
```
VDU 25, 69, 128, 2, 0, 2, 65
```

### From MOS command line (Version 1.03 or greater)

MOS also supports a VDU command which can be used to send VDU commands to the VDP.  This is useful for testing the VDP without having to write a BASIC program.  Its command is simpler than the BASIC equivalent, accepting only 8-bit integer values between 0 and 255, separated by spaces.

Example:

`VDU 17 15`: Set the text foreground colour to 15

### From Assembly code on MOS

MOS offers two ways to send VDU commands from assembly code.  The first is to use the `RST 10h` call, which will send the byte in the A register to the VDP.  The second is to use a `RST 18h` call which is used to send multiple bytes to the VDP at once.  (Neither of these calls require the string `VDU` to be included in the data sent to the VDP, and both require raw binary values to be sent, rather than an ASCII string.)

More information about these can be found in the [MOS API documentation](MOS-API.md).


## VDU Character Sequences

The aim is that the Agon's VDP should be as compatible as practical with the BBC Micro's VDU command, as well as the VDU commands supported by later versions of Acorn and R.T.Russell's BBC BASICs.  Where necessary, some extensions have been added to help facilitate the Agon's unique features and architecture.

For a more detailed description of VDU commands supported by the Agon's VDP, see [VDU Commands](VDP---VDU-Commands.md).

The following is a high-level list of the VDU sequences that are supported:

- `VDU 0`: Null (no operation)
- `VDU 1`: Send next character to "printer" (if "printer" is enabled) §§
- `VDU 2`: Enable "printer" §§
- `VDU 3`: Disable "printer" §§
- `VDU 4`: Write text at text cursor
- `VDU 5`: Write text at graphics cursor
- `VDU 6`: Enable screen (opposite of `VDU 21`) §§
- `VDU 7`: Make a short beep (BEL)
- `VDU 8`: Move cursor back one character
- `VDU 9`: Move cursor forward one character
- `VDU 10`: Move cursor down one line
- `VDU 11`: Move cursor up one line
- `VDU 12`: Clear text area (`CLS`)
- `VDU 13`: Carriage return
- `VDU 14`: Page mode On *
- `VDU 15`: Page mode Off *
- `VDU 16`: Clear graphics area (`CLG`)
- `VDU 17, colour`: Define text colour (`COLOUR`)
- `VDU 18, mode, colour`: Define graphics colour (`GCOL mode, colour`)
- `VDU 19, l, p, r, g, b`: Define logical colour (`COLOUR l, p` / `COLOUR l, r, g, b`)
- `VDU 20`: Reset palette and text/graphics colours and drawing modes §§
- `VDU 21`: Disable screen (turns of VDU command processing, except for `VDU 1` and `VDU 6`) §§
- `VDU 22, n`: [Select screen mode](VDP---Screen-Modes.md) (`MODE n`)
- `VDU 23, n`: Re-program display character / System Commands
- `VDU 24, left; bottom; right; top;`: Set graphics viewport **
- `VDU 25, mode, x; y;`: [PLOT command](VDP---PLOT-Commands.md)
- `VDU 26`: Reset graphics and text viewports **
- `VDU 27, char`: Output character to screen §
- `VDU 28, left, bottom, right, top`: Set text viewport **
- `VDU 29, x; y;`: Set graphics origin
- `VDU 30`: Home cursor
- `VDU 31, x, y`: Move text cursor to x, y text position (`TAB(x, y)`)
- `VDU 127`: Backspace

All other characters, i.e. those in the range of 32 to 126 and 128 to 255, are sent to the screen as ASCII, unaltered.

Any VDU command that is the VDP does not recognise (such as `VDU 2` when running on Quark 1.04) will be ignored.

 \* Requires VDP 1.03 or above<br>
 \** Requires VDP 1.04 or above<br>
 § Requires Console8 VDP 2.3.0 or above<br>
 §§ Requires Console8 VDP 2.5.0 or above<br>


## VDU 23 commands

`VDU 23` essentially has a split purpose.  The first is to redefine the display characters, and the second is to send commands to the VDP to access some more sophisticated behaviour.

For more information on these commands and a full list, please consult the `VDU 23` section of the [VDU Commands](VDP---VDU-Commands.md) document.  This includes the [Bitmap and Sprite API](VDP---Bitmaps-API.md).

Amongst this you will also find [system commands](VDP---System-Commands.md), which start with `VDU 23, 0` and are unique to the Agon platform.  Within the system commands set you will find the [Audio API](VDP---Enhanced-Audio-API.md), and [Buffered Commands API](VDP---Buffered-Commands-API).


