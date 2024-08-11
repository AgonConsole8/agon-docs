# VDU 23, 0, &95: Font management

From Console8 VDP 2.8.0 onwards, the VDP now has an API to allow for different fonts to be uploaded to the VDP and used on your Agon.  At this time the VDP supports mono-spaced fonts only, each only at a single size.  Future versions of this API may introduce support for variable width fonts, and displaying text at different sizes.

As with other APIs, the data for fonts is stored in buffers which are sent to the VDP using the [Buffered Commands API](Buffered-Commands-API.md).

Generally when selecting a font to use, you will need to upload the font data to a buffer on the VDP, and then indicate that buffer contains a font.  Once this has been done the font can be selected and used to draw text on the screen.  The system font will always be available, and can be used by selecting font -1 (65535).

You can define as many different fonts on the VDP as you have buffers available, and can switch between them as needed.  Your font selection is tied to your currently selected cursor, so you can select different fonts for use with your graphics cursor and your text cursor.

The commands for the font API are as follows:

### `VDU 23, 0, &95, 0, bufferId; flags`: Select font

Select a font to use for drawing text.

The `bufferId` here must refer to a buffer that has been uploaded to the VDP, and marked as a font using `VDU 23, 0, &95, 1, <args>`, as described below, or -1 (65535) for the system font.

The `flags` parameter is a bitfield which can be used to specify options for the font.  The following flags are supported:

| Bit | Description |
| --- | ----------- |
| 0   | Adjust cursor position to ensure text baseline is aligned |
| 1-7 | Reserved for future use |

If you wish to change the font within a line of text, you should use a flags setting of `1`.  This will use data within the font to ensure that the cursor position is moved to keep the text baseline aligned.

When changing the font at the start of a new line, you should use a flags setting of `0`, as otherwise the text may overlap the previous line.

### `VDU 23, 0, &95, 1, bufferId; width, height, ascent, flags`: Create font from buffer

This command is used to mark a buffer as containing font data.  The `bufferId` must refer to a buffer that has been uploaded to the VDP, and contains font data.  The `width` and `height` parameters indicate the width and height of each character in the font in pixels, and `ascent` is the distance in pixels from the top of the character to the baseline.

The `flags` parameter is reserved for future use, and should be set to 0.  In the future it will be used for indicating variable width fonts, and other font properties.

Font data is assumed to be a contiguous block of data, one character at a time, with a full 256 character definitions, from character 0 to 255.  The buffer passed to this command should contain a single block of data of an appropriate size for the font.  If the data has been uploaded in multiple parts, it should be concatenated into a single block before calling this command using the appropriate commands in the buffered commands API.

Fonts are monochrome, and the data is assumed to be one character at a time.  Each character is stored in a byte-aligned format, where each row of a character is stored in a number of bytes equal to `(width + 7) / 8`, with their pixels organised most significant bit first, with as many rows as indicated by the `height`.  "Most significant bit first" means that the leftmost pixel of the character is stored in the most significant bit of the first byte, so a font that is 6 pixels wide would be stored in bits 7-2, with bits 1 and 0 ignored when rendering the font.

### `VDU 23, 0, &95, 2, bufferId; field, value;`: Set or adjust font property

Allows you to set or adjust properties of a font that has been created using `VDU 23, 0, &95, 1, <args>`.  The `bufferId` must refer to a buffer that has been marked as containing font data.

The `field` parameter indicates the property to set, and the `value` parameter is the value to set it to.  Whilst the `value` must be sent as a 16-bit value, most of the fields will only use the least significant 8-bits of the sent value.  The following fields are supported:

| Field | Description |
| ----- | ----------- |
| 0	    | width |
| 1	    | height |
| 2	    | ascent |
| 3	    | flags |
| 4	    | buffer for character pointers (for variable width fonts) * |
| 5	    | point size (defaults to 0) * |
| 6	    | inleading (defaults to 0) * |
| 7	    | exleading (defaults to 0) * |
| 8	    | weight (defaults to 400) * |
| 9	    | character set (defaults to 255) * |
| 10    | code page (defaults to 1252) * |

Fields 0-4 directly equate to the parameters passed to `VDU 23, 0, &95, 1, <args>`.

\* Please note that as of VDP 2.8.0, whilst data can be set for these fields, setting them will not affect the rendering of the font.  They are reserved for future use.

### `VDU 23, 0, &95, 3, bufferId; [<args>]`: Reserved

This function is reserved for future use, and should not be used at this time.

(The intent is that this will be used to set the name of the font.)

### `VDU 23, 0, &95, 4, bufferId;`: Clear/Delete font

Deletes a font that has been created using `VDU 23, 0, &95, 1, <args>`.  The `bufferId` must refer to a buffer that has been marked as containing font data.

NB: This does not delete the buffer itself, only the font definition.  The data remains intact in the buffer, and can be re-used to create a new font if desired.

### `VDU 23, 0, &95, 5, bufferId;`: Copy system font to buffer

Copies the system font data into a buffer and makes a new font definition for it.

The buffer at the `bufferId` will be cleared first and then the system font data will be copied into it.  The buffer will then be marked as containing font data, and can be used as a font in the same way as any other font.

This command will let you make multiple copies of the system font which can then be modified independently of each other using the buffered commands API.

