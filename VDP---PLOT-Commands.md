# VDU 25: PLOT commands

`VDU 25, code, x; y;`

The Agon VDP system supports a number of `PLOT` commands via `VDU 25`, which are used to draw lines, circles, and other shapes on the screen.  The command set is inherited from Acorn's BBC Micro, plus Acorn's Graphics eXtension ROM (GXR).

PLOT commands sent to the VDP all require a command byte, followed by two 16-bit values for X and Y coordinates.  The command byte is used to specify the drawing operation, and the mode in which the drawing operation is performed.

Most PLOT commands require additional coordinates than the single pair provided with the command.  The graphics system works using a concept of a "graphics cursor", and will keep track of the last few positions of the graphics cursor.  Every PLOT command "pushes" a copy of the current graphics cursor position onto a stack, and then moves the graphics cursor to the new position.  Commands that require additional coordinates will look at the last few positions on the stack to determine the additional coordinates.

Support for PLOT commands has grown over time, but is still only a subset of those available on a BBC Micro with a GXR ROM.

PLOT commands are essentially split into a drawing operation, and a mode that controls how the drawing operation is performed.  There are 8 different modes for each operation.  The command byte is effectively split into two parts where the lower 3 bits define the mode, and the upper 5 bits define the drawing operation.

The complete set of PLOT codes supported by the Agon VDP follows.  For completeness, all commands from Acorn's command set are shown, including those that are not yet supported.

| PLOT code | (Decimal) | Effect |
| --------- | --------- | ------ |
| &00-&07 | 0-7 | Solid line, includes both ends |
| &08-&0F | 8-15 | Solid line, final point omitted |
| &10-&17 | 16-23 | Not supported (Dot-dash line, includes both ends, pattern restarted) |
| &18-&1F | 24-31 | Not supported (Dot-dash line, first point omitted, pattern restarted) |
| &20-&27 | 32-39 | Solid line, first point omitted |
| &28-&2F | 40-47 | Solid line, both points omitted |
| &30-&37 | 48-55 | Not supported (Dot-dash line, first point omitted, pattern continued) |
| &38-&3F | 56-63 | Not supported (Dot-dash line, both points omitted, pattern continued) |
| &40-&47 | 64-71 | Point plot |
| &48-&4F | 72-79 | Line fill left and right to non-background §§ |
| &50-&57 | 80-87 | Triangle fill |
| &58-&5F | 88-95 | Line fill right to background §§ |
| &60-&67 | 96-103 | Rectangle fill |
| &68-&6F | 104-111 | Line fill left and right to foreground §§ |
| &70-&77 | 112-119 | Parallelogram fill |
| &78-&7F | 120-127 | Line fill right to non-foreground §§ |
| &80-&87 | 128-135 | Not supported (Flood until non-background) |
| &88-&8F | 136-143 | Not supported (Flood until foreground) |
| &90-&97 | 144-151 | Circle outline |
| &98-&9F | 152-159 | Circle fill |
| &A0-&A7 | 160-167 | Not supported (Circular arc) |
| &A8-&AF | 168-175 | Not supported (Circular segment) |
| &B0-&B7 | 176-183 | Not supported (Circular sector) |
| &B8-&BF | 184-191 | Rectangle copy/move |
| &C0-&C7 | 192-199 | Not supported (Ellipse outline) |
| &C8-&CF | 200-207 | Not supported (Ellipse fill) |
| &D0-&D7 | 208-215 | Not defined |
| &D8-&DF | 216-223 | Not defined |
| &E0-&E7 | 224-231 | Not defined |
| &E8-&EF | 232-239 | Bitmap plot § |
| &F0-&F7 | 240-247 | Not defined |
| &F8-&FF | 248-255 | Not defined |

Within each group of eight plot codes, the effects are as follows:

| Plot code | Effect |
| --------- | ------ |
| 0  (8) | Move relative |
| 1  (9) | Plot relative in current foreground colour |
| 2  (A) | Plot relative in logical inverse colour §§§ |
| 3  (B) | Plot relative in current background colour |
| 4  (C) | Move absolute |
| 5  (D) | Plot absolute in current foreground colour |
| 6  (E) | Plot absolute in logical inverse colour §§§ |
| 7  (F) | Plot absolute in current background colour |

Codes 0-3 use the position data provided as part of the command as a relative position, adding the position given to the current graphical cursor position.  Codes 4-7 use the position data provided as part of the command as an absolute position, setting the current graphical cursor position to the position given.

The various "Line fill" plot commands have an additional effect, which is to adjust the graphics cursor position to the right-hand edge of the filled line, if one was drawn.

§ Support added in Agon Console8 VDP 2.1.0
§§ Support added in Agon Console8 VDP 2.2.0
§§§ Support added in Agon Console8 VDP 2.6.0


## Interaction with GCOL paint modes

The GCOL command is used to set the paint mode for the PLOT command.  The paint mode is used to control how the PLOT command interacts with the existing pixels on the screen.

Up until Console8 VDP 2.6.0 the only fully supported GCOL paint mode was mode 0, which sets the pixel to the target colour value.  This is the default mode, and is used if no GCOL command has been issued.  There was limited support for mode 4, which inverts the pixel, but this was only supported for straight line drawing operations.

As of Console8 VDP 2.6.0, the following modes are now available for all currently supported plot operations:

| Mode | Effect |
| ---- | ------ |
| 0 | Set on-screen pixel to target colour value |
| 1 | OR value with the on-screen pixel |
| 2 | AND value with the on-screen pixel |
| 3 | EOR value with the on-screen pixel |
| 4 | Invert the on-screen pixel |
| 5 | No operation |
| 6 | AND the inverse of the specified colour with the on-screen pixel |
| 7 | OR the inverse of the specified colour with the on-screen pixel

PLOT commands using an "inverse" plot code are essentially identical to setting a GCOL paint mode of 4, and will temporarily override the current GCOL paint mode if a different GCOL paint mode is set. 


## Filled triangles (PLOT codes &50-&57)

Filled triangles are drawn using the last two positions on the graphics cursor stack and the position given with the PLOT command.

The behaviour of the triangle fill command has changed slightly in Agon Console8 VDP 2.6.0, and you may observe some slight differences in the exact pixels drawn when using this command.  Unfortunately the old behaviour was not compatible with the new GCOL paint modes, and so the behaviour had to be changed.  The behaviour is consistent with general "best practice" for triangle fill algorithms, and so should be more predictable and reliable.  As an example, with the revised triangle plotting it is possible to draw a fan of triangles using an EOR paint mode and you will not see the edges of the triangles "cancel each other out" as you would have done with the old behaviour.

This new behaviour is not entirely consistent with the BBC Micro or GXR ROM, but it is more consistent with modern graphics systems and should be more predictable and reliable.  In practice, the differences are minimal and should not affect most programs.


## Rectangle copy/move (PLOT codes &B8-&BF)

These PLOT codes are interpreted slightly differently from others.  The purpose of these commands is to copy or move a rectangle of pixels from one location to another.  The rectangle is defined by the last two positions on the graphics cursor stack, and the position provided with the PLOT command is the destination to copy the rectangle to.

A "move" operation will copy the rectangle to the new location, and then clear the original rectangle using the currently defined background colour.  (NB the background GCOL paint mode is _not_ used when filling the original rectangle - the pixels will be set with the background colour).  A "copy" operation will copy the rectangle to the new location, but leave the original rectangle in place.

| PLOT code | Effect |
| --------- | ------ |
| &B8 | Move cursor relative to last position |
| &B9 | Relative rectangle move |
| &BA | Relative rectangle copy |
| &BB | Relative rectangle copy |
| &BC | Move cursor to absolute position |
| &BD | Absolute rectangle move |
| &BE | Absolute rectangle copy |
| &BF | Absolute rectangle copy |


## Bitmap plots (PLOT codes &E8-&EF)

Bitmap plots will draw the currently selected bitmap to the screen.

Before a bitmap plot can be used a valid bitmap must be selected using either `VDU 23, 27, 0, id` or `VDU 23, 27, &20, bufferId;`.

Bitmap plots will only draw non-transparent pixels to the screen.  When using "foreground" PLOTs, the bitmap is drawn to screen using the current foreground GCOL painting mode using the colour from the bitmap.  When using "background" PLOTs, the bitmap is drawn to screen using the current background GCOL painting mode using the currently selected background colour.  Inverse plot modes invert on-screen pixels that correspond to non-transparent pixels in the bitmap.


## PLOT support prior to VDP 1.04

Prior to VDP 1.04 the PLOT command support was very limited, did not support relative positioning, and had buggy and incorrect interpretations of some of the PLOT codes.  As of VDP 1.04 the PLOT command support has been greatly improved, and is now much more compatible with the BBC Micro and GXR ROM.

The most significant bug was the mis-interpretation of several "move" commands to instead be "draw".  As a result, some programs written for earlier versions of the VDP firmware may no longer work correctly.  Fixing these programs however is usually very straightforward and usually just involves changing the PLOT code used.


## Compatibility

If the VDP does not recognise a plot operation, it will be ignored.  This can allow you to write programs that "feature detect" whether the VDP supports a particular plot operation, and then use it if it does, or fall back to an alternative if it does not.
