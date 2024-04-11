# Screen Modes

The Agon VDP supports a number of different screen modes, which are listed below.

Screen modes are selected using `VDU 22, mode`, or in BASIC using `MODE mode`.

The list of screen modes available for use got revised and greatly expanded in VDP version 1.04.  The revision was mostly to make the screen modes more compatible with the VGA standards, and thus more compatible with modern monitors.  The original screen modes are still available, but are now considered legacy modes.

It should be noted that whilst the Agon VDU command system is designed to be compatible with the BBC Micro, the screen modes are not.  The BBC Micro has a different set of screen modes, and the Agon VDP does not support them.

By default, the Agon VDU system uses and adopts the BBC Micro's "logical" coordinate system for graphics, which means that each screen mode appears to be 1280x1024 to software with the graphics origin 0,0 point at the bottom left of the screen.  As a result programs written the BBC Micro can usually be fairly easily adapted.

This logical coordinate system differs from the BBC Micro and later Acorn systems in that Acorn systems all used round numbers for the multiplier between the logical and physical coordinate systems.  (This meant that on later Acorn systems some screen modes used logical resolutions other than 1024x1280, depending on the physical resolution of the mode.)  On Agon systems, the multiplier for the vertical resolution is, essentially never a round number.

To work around the limitations of the logical coordinate system, it is possible to change the graphics system to use physical coordinates instead of logical coordinates.

There are two VDU commands that will affect the screen modes that are available, and how the drawing system works.

## VDU Commands

### `VDU 22, mode`

This command selects a screen mode, where the screen mode is a single byte value and must be a mode from the list below.

Screen modes numbered above 128 are double-buffered, meaning that the screen is drawn to an off-screen buffer, and then the buffer is copied to the screen.  This prevents flickering when drawing to the screen.

As drawing operations, when in a double-buffered mode, are only performed in the off-screen buffer this means that the screen will not be updated until the active buffer is swapped.  This means, for instance, that if you enter a double-buffered screen mode in BASIC by using `MODE 129` then commands that you type will not be visible, as your input is being written to the off-screen buffer.  Commands will still be processed, but you will only see their effect when the buffer is swapped.  Buffers are swapped using `VDU 23, 0, &C3`

It may sometimes not be possible to change into a screen mode, for instance because the VDP no longer has enough memory to support the mode.  In this case the system will fall back to the current mode, and if that fails to the default mode, which is mode 1.  All modes should always be available after a reset.

### `VDU 23, 0, &C0, n`

Turn logical screen scaling on and off, where 1=on and 0=off.

When logical scaling is turned off, the graphics system will no longer use the 1280x1024 logical coordinate system and instead use pixel coordinates.  The screen origin point at 0,0 will change to be the top left of the screen, and the Y axis will go down the screen instead of up.  

Support for this was added in VDP 1.03.


### `VDU 23, 0, &C1, n`

Switch legacy modes on or off.

By default, the original screen modes 0-4 are not available and are instead replaced by new modes that are more compatible with modern monitors.  For compatibility with older software, written for Agon systems running earlier versions of the VDP firmware, this command can be used to switch back to those original, legacy, screen modes.

Support for this was added in VDP 1.04


### `VDU 23, 0, &C3`

Swap the screen buffer (double-buffered modes only) or wait for VSYNC (all modes).

This command will swap the screen buffer, if the current screen mode is double-buffered, doing so at the next VSYNC.  If the current screen mode is not double-buffered then this command will wait for the next VSYNC signal before returning.  This can be used to synchronise the screen with the vertical refresh rate of the monitor.

Waiting for VSYNC can be useful for ensuring smooth graphical animation, as it will prevent tearing of the screen.

(In BASIC performing a `*FX 19` command will also wait for VSYNC, but will not swap the screen buffer.)

Support for this was added in VDP 1.04


## Screen modes

Modes over 128 are double-buffered

### From Version 1.04 or greater

| Mode | Horz | Vert | Cols | Refresh |
|-----:|-----:|-----:|-----:|--------:|
|    0 |  640 |  480 |   16 |    60hz |
|  * 1 |  640 |  480 |    4 |    60hz |
|    2 |  640 |  480 |    2 |    60hz |
|    3 |  640 |  240 |   64 |    60hz |
|    4 |  640 |  240 |   16 |    60hz |
|    5 |  640 |  240 |    4 |    60hz |
|    6 |  640 |  240 |    2 |    60hz |
| ** 7 |  n/a |  n/a |   16 |    60hz |
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

\* Mode 1 is the "default" mode, and is the mode that the system will use on startup.  It is also the mode that the system will fall back to use if it was not possible to change to the requested mode.

\** Mode 7 is the "Teletext" mode, and essentially works in a very similar manner to the BBC Micro's Teletext mode, which was also mode 7.


### Legacy modes (prior to 1.04)

| Mode | Horz | Vert | Cols | Refresh |
|-----:|-----:|-----:|-----:|--------:|
| 0    | 1024 |  768 |    2 | 60hz    |
| 1    |  512 |  384 |   16 | 60hz    |
| 2    |  320 |  200 |   64 | 75hz    |
| 3    |  640 |  480 |   16 | 60hz    |
