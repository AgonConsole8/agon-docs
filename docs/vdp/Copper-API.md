# The "Copper" API

As of VDP 2.12.0 the VDP supports a set of functionality inspired by "Copper" effects on the Commodore Amiga.  The feature requires that a feature flag be set to enable the Copper API, which can be done using `VDU 23, 0, &F8, &310; 0;`

These effects work on screen modes that use a palette for generating their video output, i.e. 2, 4 and 16 color modes.  They are not supported in 64 color modes, as those modes have a fixed output palette.

On palette-based screen modes, the screen framebuffer is converted on the fly whilst the VGA scanout is in progress, one row at a time, into the VGA output signal.  Up to and including VDP 2.11.0 only a single palette would be used for the signal output.  As of VDP 2.12.0, the Copper API allows for multiple palettes to be defined, and a "signal list" set to define which palettes should be used for scanout for different rows of the screen.

Through the use of this API, it is possible to potentially show all 64 colours that the Agon is capable of outputting on a single screen when in a 2, 4 or 16 colour mode.

Essentially, all drawing operations in palette-based screen modes are performed using the primary palette, palette ID 0.  The existing palette changing operations will only affect the primary palette.

The Copper API supports 16-bit palette IDs, so in principle up to 65536 palettes could be defined.  However in practice you should never define that many.  These palettes are allocated in internal memory on the VDP, so there are practical limits to how many palettes can be defined.

On changing screen mode, the signal list is reset and all palettes are cleared, leaving only the default palette.

When combined with the newly introduced VSYNC callback mechanism, it is possible to automatically change the signal list on a per-frame basis, allowing for a wide range of effects to be achieved.  This can be done completely on the VDP without your main program needing to send any additional commands once it has set things up.


## Copper API Commands

### Command 0: Create a palette

`VDU 23, 0, &C4, 0, <paletteID>;`

This command creates a new palette with the specified ID.  The palette will initially be a copy of palette 0.

NB this command may fail if the VDP runs out of internal memory.

### Command 1: Delete a palette

`VDU 23, 0, &C4, 1, <paletteID>;`

Removes the palette with the specified ID.  If the palette is currently in use in the signal list, it will be removed from the signal list, with its entries replaced with palette 0.

Calling this command with palette ID 0 will have no effect.

If you call this command with palette ID 65535 then all palettes will be deleted.

### Command 2: Set a palette entry

`VDU 23, 0, &C4, 2, <paletteID>; <index>, <red>, <green>, <blue>`

Changes the palette entry in the specified palette to the specified RGB value.

The index value is the index of the palette entry to change.  This will wrap around if it is greater than the number of entries in the palette.  A 16 colour screen mode has entries numbered 0-15, a 4 colour screen mode has entries numbered 0-3, and a 2 colour screen mode has entries numbered 0-1.  Attempting to set a palette entry at index 17 will set the entry at index 1, and so on.

The colour values for this command are 8-bit values, with 0 being the darkest and 255 being the brightest, i.e. in RGB888 format, consistent with VDU 19.

If this command is used with a palette ID that does not yet exist then it will attempt to create a new palette with the specified ID.

### Command 3: Set/Update the signal list

`VDU 23, 0, &C4, 3, <bufferId>;`

This command is used to set the signal list.  The source list must be stored in a buffer, and the buffer ID is passed as an argument to this command.  Only the first block in a buffer is used.

A signal list can be updated at any time, and the new signal list will take effect immediately.

The [buffered commands API](Buffered-Commands.md) can be used to create a buffer and fill it with a signal list.

A source signal list is a set of pairs of 16-bit values stored in a buffer, where the first value is a count of rows, and the second value is the palette ID to use for those rows.  A single entry is therefore 4 bytes long.  The 16-byte values must be stored in the buffer in little-endian format.

A source signal list can be of any length, entries are read sequentially from the source buffer.  If the total number of rows in the signal list is less than the number of rows in the screen, then the last palette in the signal list will be used for the remaining rows.  If the list has more entries than there are rows on the screen then the extra items in the signal list will be ignored.

The output signal list will remain in effect until it is replaced by a new signal list, or until it is reset.  You are therefore free to do whatever you like with the buffer after the signal list has been set.

By setting a signal list, and then adjusting the source buffer to contain different values, and then re-setting the signal list you can achieve a wide range of effects.  When coupled with the VSYNC callback mechanism these effects can be run completely within the VDP without needing to involve your main program.

### Command 4: Reset the signal list

`VDU 23, 0, &C4, 4`

This command resets the signal list to the default state, where the default palette is used for all rows.

## Using copper to show all 64 colours

As noted above, the Copper API allows for multiple palettes to be defined, and a signal list set to define which palettes should be used for scanout for different rows of the screen.  This therefore allows access to all 64 colours in screen modes that otherwise would only show 2, 4 or 16 colours.

All drawing operations to the framebuffer are performed as normal, using only the primary palette for colour lookups.

If you wish to show a bitmap image in a 16 colour screen mode in a manner similar to the Amiga's "Hold and Modify" mode, you can use the Copper API to set up a signal list that will show all 64 colours on the screen.  The bitmap data however will be drawn as if the primary palette is the only palette in use.  You would therefore need to prepare your bitmap using pixel values that correspond to the primary palette entries, and then prepare palettes and a signal list to change the colours during scan-out.

## Interactions with sprites

Software sprites are drawn into the framebuffer using the primary palette, and so their appearance on-screen will be affected by the signal list.

Hardware sprites on the other hand are not drawn into the framebuffer, and so are not affected by the signal list.  They are shown in their natural colours using the Agon's available 64 colours.
