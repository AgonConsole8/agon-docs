# VDU 25: PLOT commands

The Agon VDP system supports a number of `PLOT` commands via `VDU 25`, which are used to draw lines, circles, and other shapes on the screen.  The command set is inherited from Acorn's BBC Micro, plus Acorn's Graphics eXtension ROM (GXR).

Support for PLOT commands has grown over time, but is still only a subset of those available on a BBC Micro with a GXR ROM.

Prior to VDP 1.04 the PLOT command support was very limited, did not support relative positioning, and had buggy and incorrect interpretations of some of the PLOT codes.  As of VDP 1.04 the PLOT command support has been greatly improved, and is now much more compatible with the BBC Micro and GXR ROM.

The most significant bug was the mis-interpretation of several "move" commands to instead be "draw".  As a result, some programs written for earlier versions of the VDP firmware may no longer work correctly.  Fixing these programs however is usually very straightforward and usually just involves changing the PLOT code used.

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

§ Support added in Agon Console8 VDP 2.1.0
§§ Support added in Agon Console8 VDP 2.2.0

Within each group of eight plot codes, the effects are as follows:

| Plot code | Effect |
| --------- | ------ |
| 0 | Move relative |
| 1 | Plot relative in current foreground colour |
| 2 | Not supported (Plot relative in logical inverse colour) |
| 3 | Plot relative in current background colour |
| 4 | Move absolute |
| 5 | Plot absolute in current foreground colour |
| 6 | Not supported (Plot absolute in logical inverse colour) |
| 7 | Plot absolute in current background colour |

Codes 0-3 use the position data provided as part of the command as a relative position, adding the position given to the current graphical cursor position.  Codes 4-7 use the position data provided as part of the command as an absolute position, setting the current graphical cursor position to the position given.

Codes 2 and 6 on Acorn systems plot using a logical inverse of the current pixel colour.  These operations cannot currently be supported by the graphics system the Agon VDP uses, so these codes are not supported.  Support for these codes may be added in a future version of the VDP firmware.


## Compatibility

If the VDP does not recognise a plot operation, it will be ignored.  This can allow you to write programs that "feature detect" whether the VDP supports a particular plot operation, and then use it if it does, or fall back to an alternative if it does not.
