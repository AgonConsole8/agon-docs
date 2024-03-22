# VDU 23, 27: Bitmaps, sprites, and mouse cursor

VDU 23, 27 is reserved for the bitmap, sprite, and mouse cursor functionality.

As of VDP 1.04, the bitmap system is integrated with the [Buffered Commands API](VDP---Buffered-Commands-API.md).  Bitmap data is stored in buffers, and can be manipulated using the Buffered Commands API on the VDP.


## Bitmaps

The bitmap system we have on the Agon uses commands that are inspired by Acorn's Graphics eXtension ROM (GXR) system.  Some mistakes were made in the original interpretation of the GXR commands, and so until the Console8 VDP 2.2.0 release it was not possible to use code written for GXR on the Agon.

Acorn only actually had two VDU commands for what it called "sprites", which in the Agon we consider to be "bitmaps".  The first command was used to select a bitmap (so that it may later be used with an appropriate PLOT command), and the second was to define a bitmap from an area of screen.

The approach taken on Agon (initially at least) was to redefine the "define bitmap from screen" command to instead allow the uploading of a binary bitmap image.  In doing so, the parameters of the command changed, and the bitmap identifier was lost from the command parameters.  Instead, on the Agon, you need to always perform a "select bitmap" command before any other bitmap commands to set the bitmap being used.  Additionally on the Agon, prior to Console8 VDP 2.2.0, plotting bitmaps could only be performed with a custom command, and not with the standard `PLOT` commands.

As of Console8 VDP 2.2.0, it is now possible to use Acorn GXR style "sprite" code on an Agon.  The `PLOT` code for drawing bitmaps is now supported, and bitmap command 1 can now be used to capture screen data into a bitmap identically to the GXR.  The documentation on the [`PLOT` command](VDP---PLOT-Command.md) (`VDU 25`) explains how to use it to draw bitmaps.

In addition to GXRs two commands, the Agon VDP has several other commands for managing bitmaps, and additional commands to manage "sprites".

As has been noted above, bitmap data is stored in buffers.  Acorn's original API design only allows for a maximum of 255 bitmaps, as they used an 8-bit ID, however buffers are identified with a 16-bit identifier.  This means that the Agon VDP can support a much larger number of bitmaps, in theory up to 65534 of them.  To allow access to these additional bitmaps, the Agon VDP several additional commands working with bitmaps using 16-bit IDs.  Those 16-bit IDs directly correspond to the IDs of the buffers in which the bitmpas are stored.  Bitmaps with 8-bit IDs are automatically mapped onto buffer IDs in the range 64000-64255.

In general, commands to manage bitmaps using 16-bit IDs are numbered 32 higher than the equivalent command using an 8-bit ID (32 is `&20` in hexadecimal).  For instance, the command to select a bitmap with an 8-bit ID is `VDU 23, 27, 0, n`, whereas the command to select a bitmap with a 16-bit ID is `VDU 23, 27, &20, bufferId;`.

Storing bitmaps in buffers allows for their data to be manipulated using the buffered commands API.  This can, for instance, allow for the colours of bitmaps to be changed, or for a bitmap to be mirrored, or split into multiple bitmaps.

\* Commands marked with an asterisk are only available in VDP 2.2.0 or later.
\** Commands marked with a double asterisk are only available in VDP 2.6.0 or later.

The commands to manage bitmaps are as follows:

### `VDU 23, 27, 0, n`: Select bitmap n

This selects the bitmap with the given 8-bit ID as being the currently "active" bitmap for subsequent bitmap commands.

As noted above, bitmaps with 8-bit IDs are stored in buffers with an ID of 64000+`n`.

You need to select a bitmap before it can be plotted onto the screen or use any other bitmap commands.


### `VDU 23, 27, 1, w; h; b1, b2 ... bn`: Load colour bitmap data into current bitmap

This command is used to load bitmap data directly into the currently selected bitmap.

Before you can load bitmap data into a bitmap, you must first select the bitmap using `VDU 23, 27, 0, n`, where `n` is the 8-bit ID of the bitmap to be loaded, or using `VDU 23, 27, &20, bufferId;` using a 16-bit buffer ID.

The bitmap data is given as a series of bytes as part of the command in RGBA8888 format, meaning that each pixel is represented by four bytes, one each for the red, green, blue, and alpha components of the pixel.  Each component value is in the range of 0-255.  The bytes are given in row-major order, starting at the top left of the bitmap, and working across the rows, and then down the columns.

The width and height of the bitmap must be given in pixels, and must match the number of bytes given in the data.  If the width and height do not match the data then the command will fail.

It should be noted that whilst the image format supported by this command is for full 24-bit colour images with 256 levels of transparency (alpha) per pixel, the Agon hardware is only capable of displaying 2-bits per colour channel.  The graphics system also does not really support transparency, so any non-zero alpha value is interpreted as "fully visible".  Data loaded via this command remains in RGBA8888 format on the VDP, but is converted on the fly when the bitmap is drawn to the screen.

(As RGBA8888 is a very wasteful format, given the hardware limitations, other options are now available.  Loading bitmaps with this command can take a long time, and it is not possible to intersperse other commands, such as showing progress on-screen, whilst the data is being sent.  There are however alternative ways of managing the data which avoids these issues.  Please see the section on "Using buffers for bitmaps" in the [Buffered Commands API](VDP---Buffered-Commands-API.md) document for more information.)

### `VDU 23, 27, 1, n, 0, 0;`: Capture screen data into bitmap n *

This command, added in Agon Console8 VDP 2.2.0, is compatible with the Acorn GXR command of the same number.  It captures the screen data into the bitmap with the given 8-bit ID.

This version of the `VDU 23, 27, 1` command differs from the one documented before in that you do not need to explicitly select a bitmap before capturing screen data into it.  The bitmap ID is passed as a parameter to this version of the command.

As you may have noticed, this command does not include any dimensions for the bitmap.  Those are set via the graphics system, taking the last two graphics cursor positions to define the bitmap rectangle.  Any `PLOT`, or `VDU 25`, style command will push the graphics cursor position - typically "move" style plot commands are used to define the rectangle.

To be clear, this command should be performed _after_ two "move" style PLOT commands.

If a bitmap with the given ID already exists then it will be overwritten, and similarly if a buffer was already defined with the ID 64000+`n` then that will be overwritten too.

Up to and including the Console8 VDP 2.5.0 release, the bitmap data captured using this command will be stored in "native" format.  The nature of the "native" format varies depending on the screen mode.  For all screen modes the bitmap will use 1 byte per pixel, but the data within that byte varies.  In 64 colour modes, the data is essentially in RGB222 with no alpha channel.  It is possible to convert bitmaps captured in 64 colour modes to RGBA2222 format by using the OR operation of the "adjust" command from the [Buffered Commands API](VDP---Buffered-Commands-API.md), ORing all the bytes in the bitmap's buffer with `&C0` to set the pixel alpha bits to be "opaque", and then re-creating the bitmap from the corresponding buffer to be RGBA2222 (format 1).  For all other screen modes, the byte represents a palette index.  An unfortunate effect of this is that a bitmap captured in one screen mode may not be compatible with other screen modes.

Also up to and including the Console8 VDP 2.5.0 release, bitmaps captured with this command would use "exclusive" coordinates, and so would be 1 pixel shorter and narrower than the area defined by the graphics cursor positions.  (This is _not_ consistent with the behaviour of this command on Acorn systems.)

From Console8 VDP 2.6.0, the bitmap data captured using this command will be stored in RGBA2222 format, regardless of the screen mode.  This is to ensure that the data is consistent and predictable, and to allow for the use of the bitmap in any screen mode.

Bitmaps captured on Console8 VDP 2.6.0 or later also now use "inclusive" coordinates, and so will be 1 pixel taller and wider than bitmaps captured on earlier versions of the VDP.  This is to ensure that the bitmap captures the entire area defined by the graphics cursor positions.  This is consistent with the behaviour of this command on Acorn systems.

### `VDU 23, 27, 2, w; h; col1; col2;`: Create a solid colour rectangular bitmap 

Creates a new bitmap in RGBA8888 format with the given width and height, and fills it with a solid colour.  The colour is given as two 16-bit numbers, col1 and col2, which are combined to form a 32-bit number in RGBA colour range.

The bitmap is created in the buffer with the ID 64000+`n`, where `n` is the 8-bit ID given in the command.

### `VDU 23, 27, 3, x; y;`: Draw current bitmap on screen at pixel position x, y

Prior to Agon Console8 VDP 2.2.0, this was the only way to draw a bitmap on-screen.  On more up to date VDP versions is strongly recommended that you use the appropriate [PLOT command](VDP---PLOT-Commands.md) instead.

Before this command can be used, a bitmap must be selected using `VDU 23, 27, 0, n`, where `n` is the 8-bit ID of the bitmap to be drawn, or using `VDU 23, 27, &20, bufferId;` using a 16-bit buffer ID.

The x and y parameters give the pixel position on the screen at which the top left corner of the bitmap will be drawn.  This is in contrast to `PLOT` commands which will (by default) use OS Coordinates, where the origin is at the bottom left of the screen and the screen is always considered to have the dimensions 1280x1024.

Please note that this command does not obey the current graphics viewport or the currently selected coordinate system.  The bitmap will be drawn at the given pixel position, and will _not_ be clipped by the viewport.  To draw bitmaps with clipping, you are advised to use the appropriate bitmap [PLOT commands](VDP---PLOT-Commands.md) instead.

### `VDU 23, 27, &20, bufferId;`: Select bitmap using a 16-bit buffer ID *

This command is essentially identical to `VDU 23, 27, 0, n`, however it uses a 16-bit buffer ID instead of an 8-bit bitmap ID.

### `VDU 23, 27, &21, w; h; format`: Create bitmap from selected buffer *

This command creates a new bitmap from the data in the currently selected buffer.

Whilst bitmap data can be loaded into a buffer at any time, before that data can drawn on-screen it must be converted into a bitmap.  The VDP needs to understand the width and height of the bitmap, and the format of the data.  This command performs that conversion.

Valid values for the format parameter are:

| Value | Meaning |
| ----- | ------- |
| 0 | RGBA8888 (4-bytes per pixel) |
| 1 | RGBA2222 (1-bytes per pixel) |
| 2 | Mono/Mask (1-bit per pixel) |
| 3 | Reserved for internal use by VDP ("native" format) |

It should be noted that the "alpha" channel in both the RGBA8888 and RGBA2222 formats is not properly used by the Agon VDP.  Any non-zero value in the alpha channel is interpreted as "fully visible".  The Agon VDP does not currently support transparency.  The alpha channel is still stored in the bitmap data, and is used when the bitmap is drawn to the screen, but it is not used to blend the bitmap with the background.  (If we improve support for transparency in the future, to maintain compatibility we will do so by adding new bitmap formats, rather than changing the behaviour of the existing formats.)

Mono/Mask bitmaps can be of any width, but their data must be sent using a whole number of bytes per row.  Mon/mask bitmaps are also required to have a colour, and will use the currently selected graphics foreground colour.  If you wish to use a different colour then you must change the graphics foreground colour before creating the bitmap.  Mono/mask bitmaps only draw their "on" pixels, and the "off" pixels are transparent.

The use of the "native" format is reserved for internal use by the VDP.  They have some significant limitations, and are not intended for general use.

As with `VDU 23, 27, 1, w; h; b1, b2...bn`, the width and height of the bitmap must be given in pixels, and must match the number of bytes given in the data.  If the width and height do not match the data then the command will fail.

Prior to the Console8 VDP 2.2.0 release, mono/mask bitmaps were not supported.

### `VDU 23, 27, &21, bitmapId; 0;`

This command captures a bitmap from the screen into the buffer with the given 16-bit buffer ID.  It works the same as `VDU 23, 27, 1, n, 0, 0;`, but using a 16-bit buffer ID instead of an 8-bit bitmap ID.

Support for this command was added in VDP 2.2.0.


## Sprites

The Sprites system on the Agon VDP is an extension to the bitmap system.  They are not true "hardware sprites" as one might find on some 8-bit machines or consoles.  They are best thought of as "automatically drawn bitmaps".

Technically the VDP can support up to 256 sprites.  They must be defined contiguously, and so the first sprite is sprite 0.  (In contrast, bitmaps can have any ID from 0 to 65534.)  Once a selection of sprites have been defined, you can activate them using the `VDU 23, 27, 7, n` command, where `n` is the number of sprites to activate.  This will activate the first `n` sprites, starting with sprite 0.  All sprites from 0 to n-1 must be defined.

A single sprite can have multiple "frames", referring to different bitmaps.  (These bitmaps do not need to be the same size.)  This allows a sprite to include an animation sequence, which can be stepped through one frame at a time, or picked in any order.

Any format of bitmap can be used as a sprite frame.  It should be noted however that "native" format bitmaps are not recommended for use as sprite frames, as they cannot get erased from the screen.  (As noted above, the "native" bitmap format is not really intended for general use.)  This is part of why from Agon Console8 VDP 2.6.0 bitmaps captured from the screen are now stored in RGBA2222 format.

An "active" sprite can be hidden, so it will stop being drawn, and then later shown again.

Moving sprites around the screen is done by changing the position of the sprite.  This can be done either by setting the absolute position of the sprite, or by moving the sprite by a given number of pixels.  (Sprites are positioned using pixel coordinates, and not by the logical OS coordinate system.)  In the current sprite system, sprites will not update their position on-screen until either another drawing operation is performed or an explicit `VDU 23, 27, 15` command is performed.

Here are the sprite commands:

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
- `VDU 23, 27, 18, n`: Set the current sprite GCOL paint mode to n **
- `VDU 23, 27, &26, n;`: Add bitmap n as a frame to current sprite using a 16-bit buffer ID

### Notes on sprites

This advice is valid up to and including the Agon Console8 VDP 2.6.0 release.  In the future this may change.

The nature of how sprites are currently supported in the Agon VDP means that performance is not as good as it could be, and so they should be used sparingly.  If you have too many sprites on-screen then you may notice that some sprites may flicker, especially if they are positioned closer to the top of the screen.  Sprites are currently always drawn in order, so those with lower IDs will flicker less than those with higher IDs.

Performance of the Agon emulator differs significantly from real hardware, owing to how the VDP system is emulated.  You will be able to have more sprites on-screen on the emulator than you will on real hardware.

Technically, sprites are implemented as "automatically drawn bitmaps".  When a sprite is drawn on-screen, a copy of the area of screen that it covers is made, and then the sprite bitmap is drawn over that screen area.  (Sprite bitmaps can be transparent, and the background will show through.)  When a sprite is hidden, the area of screen that it covered is copied back onto the screen.  When you have a lot of sprites, this can mean that a lot of screen data is being copied around.

Owing to inefficiencies in how sprites have been implemented, _any_ operation that draws something to the screen will cause _all_ sprites to be hidden, the drawing operation performed, and then all sprites are redrawn.  (This includes, for example, the flashing of the text cursor.)  Sprites are redrawn in order by their ID at the beginning of the screen being drawn to the VGA port.  This means that if there are a lot of active sprites, those later in the list get drawn to the screen buffer later, possibly after that part of the screen has been sent out of the VGA port.  This, essentially, is the cause of the flickering.

One way to reduce flickering is to try to avoid using sprites near the top of the screen.  This gives more opportunity for the sprites to be redrawn.

Owing to how sprites work, it is hard to offer definitive advice on how many sprites you can use.  The size of sprites you are using will also be a factor.  It is best to experiment and see what works for you.

Sprites also behave differently on double-buffered screen modes.  On those modes sprites are never "hidden" and are instead drawn to screen before a buffer swap.  This means they tend not to flicker, but care needs to be taken to ensure that the screen is properly redrawn.

Up to and including the Console8 VDP 2.5.0 release, sprites using bitmaps captured from screen would not work properly, and would not be erased from the screen.  This is because the bitmaps were stored in "native" format, and the VDP did not have the ability to erase them.  This has been fixed in the Console8 VDP 2.6.0 release, and bitmaps captured from the screen are now stored in RGBA2222 format, and so can be erased from the screen.

Console8 VDP 2.6.0 also introduces the ability to set the GCOL paint mode for sprites.  This can allow for more sophisticated drawing operations to be performed with sprites, and some interesting effects.

For completely optimal graphical performance, it is usually best to avoid using sprites, and instead use the bitmap system directly.  Using a small number of sprites can be a reasonable compromise.


## Mouse cursor

### `VDU 23, 27, &40, hotX, hotY`: Setup a mouse cursor

Sets up a new mouse cursor using the currently selected bitmap, with a hotspot at hotX, hotY.

Once a mouse cursor has been set up in this way, it can be selected for display using `VDU 23, 0, &89, 3, bitmapId;`.  Please note that this is a 16-bit ID, therefore if a bitmap had been selected using an 8-bit identifier you will need to add 64000 to the ID to get the 16-bit ID.

Care should be taken to avoid using bitmapIds from 0-18, as those conflict with existing cursor Ids.
