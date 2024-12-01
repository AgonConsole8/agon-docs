# Tile Engine

The Tile Engine is a series of routines in the VDP that allows for the loading of tile graphic data into a "Tile Bank" and the defining of a "Tile Map" that can be displayed as a "Tile Layer" by the Agon. Commands are provided that allow the tile map to be scrolled horizontally and/or vertically on a per-pixel basis, controlled by any Agon language that supports the VDU interface.

The Tile Engine is accessed through the ```VDU 23,0,&C2``` (aka ```VDU 23,0,194```) commands. While not all VDU commands starting with this sequence are currently in use, many are defined and reserved for future enhancements.


## Drawing Co-ordinates

As the Tile Engine uses 8x8 character tiles, the Acorn text co-ordinate system has been adopted for specifying the position of a tile on screen with (0,0) at the top left of the screen. However, as this is limited to eight pixel blocks, the Tile Engine can additionally take an X and Y offset parameter to provide per-pixel positioning within the specified 1block.


## Tile Bank Commands

The Tile Bank is a block of memory in the VDP that can hold up to 255 8x8 character tiles in the RGBA2222 format.

### Initialise/Reset Tile Bank

```VDU 23, 0, &C2, 0, 0, 0, 0, 0```

This command will initialise or reset (clear) the tile bank. It must be executed before attempting to load any tiles.

### Load Tile Into Bank

```VDU 23, 0, &C2, 1, <tileid>, b1, b2 ... b64```

This command will load 64 bytes of data into the specified tile id.

Valid values for ```tileid``` are 1-255. Do not load any data into Tile Id 0 as it has a special purpose and will not be drawn.

#### Tile Format

Tiles are 8x8 pixel characters defined using the RGBA2222 format, where each pixel occupies one byte of memory. Due to hardware limitations, the Agon Light can display a maximum of 64 colours.

The actual format of the RGBA2222 pixel is AABBGGRR, where each byte has two bits for each channel:

- A = Alpha (transparency)
- B = Blue
- G = Green
- R = Red

For the Red, Green and Blue colours, there are two bits per channel, enabling four shades (values 00, 01, 10, 11) per channel and totalling the 64 colours available on the Agon.

Although there are two bits for the Alpha channel, there are only two transparency states implemented on the Agon:

- If AA = 11 then pixel is opaque
- If AA = 10 then pixel is opaque
- If AA = 01 then pixel is opaque
- If AA = 00 then pixel is transparent

The Tile Engine requires that opaque RGBA2222 pixels have the Alpha channel bits set to 11 followed by the colour values in BBGGRR order. Transparent pixels should have the Alpha channel bits set to 00 and the RGB colour values for all other channels set to 00.

The position of each byte in the tile is shown below (in hex), working from top left to bottom right in rows:

```
00 01 02 03 04 05 06 07
08 09 0A 0B 0C 0D 0E 0F
10 11 12 13 14 15 16 17
18 19 1A 1B 1C 1D 1E 1F
20 21 22 23 24 25 26 27
28 29 2A 2B 2C 2D 2E 2F
30 31 32 33 34 35 36 37
38 39 3A 3B 3C 3D 3E 3F
```


### Draw Tile From Tile Bank

```VDU 23, 0, &C2, 6, 0, <tileid>, 0, <xpos>, <ypos>, <xoffset>, <yoffset>, <attribute>```

This command allows for a single tile to be drawn on screen.

### Delete/Free Tile Bank

```VDU 23, 0, &C2, 7, 0```

This command will delete the Tile Bank (if allocated) and free up previously allocated memory in the ESP32.

## Tile Map Commands

The Tile Map is an array that contains a one-byte Tile ID and an associated Attribute byte.

### Tile ID

The ```<tileid>``` refers to the specified tile in the Tile Bank. Values between 0-255 are valid. If a ```<tileid>``` is set to 0, the tile is not drawn and the existing background pixels are untouched.

### Tile Attribute

The ```<attribute>``` byte provides additional instructions on how to display the tile. The attribute byte is configured as follows:

| Bit | Description                     |
|-----|---------------------------------|
| 7   | Reserved - should be set to 0   |
| 6   | Reserved - should be set to 0   |
| 5   | Reserved - should be set to 0   |
| 4   | Reserved - should be set to 0   |
| 3   | Reserved - should be set to 0   |
| 2   | Reserved - should be set to 0   |
| 1   | Flip Y:  0 = No flip, 1 = flip  |
| 0   | Flip X:  0 = No flip, 1 = flip  |

If the ```<tileid>``` is set to 0, the attribute byte should also be set to 0 in order to ensure that the tile is not drawn.


### Initialise/Reset Tile Map

```VDU 23, 0, &C2, 16, 0, <tilemapsize>, 0, 0```

This command is used to initialise the Tile Map in the VDP. The size of the tile map is configurable with the following options for ```<tilemapsize>``` allowed:

| Tile Map Mode   | Tile Map Size |
|-----------------|---------------|
| 0               | 32x32         |
| 1               | 32x64         |
| 2               | 32x128        |
| 3               | 64x32         |
| 4               | 64x64         |
| 5               | 64x128        |
| 6               | 128x32        |
| 7               | 128x64        |
| 8               | 128x128       |

### Set Tile Properties

```VDU 23, 0, &C2, 17, 0, <xpos>, <ypos>, <tileid>, <attribute>```

This command writes the ```<tileid>``` and ```<attribute>``` bytes to the specified X/Y position in the Tile Map.


### Delete/Free Tile Map

```VDU 23, 0, &C2, 23, 0```

This command deletes and frees up memory allocated to the Tile Map.


## Tile Layer Commands

The Tile Layer is the viewport into which the Tile Map is displayed. 


### Initialise/Reset Tile Layer

```VDU 23, 0, &C2, 24, 0, <tilelayersize>, 0, 0```

This command initalises or resets the Tile Layer. It should be configured to match the size of the required screen mode (this behaviour is not enforced, but the display may appear odd with the layer in only a part of it).


| Tile Layer Mode | Tile Layer Size | Notes             |
|-----------------|-----------------|-------------------|
| 0               | 80x60           | For 640x480 modes |
| 1               | 80x30           | For 640x240 modes |
| 2               | 40x30           | For 320x240 modes |
| 3               | 40x25           | For 320x200 modes |



## Set Tile Layer Scroll Parameters

```VDU 23, 0, &C2, 26, 0, <xpos>, <ypos>, <xoffset>, <yoffset>```

This command specifies the X and Y positions in the Tile Map to start drawing the layer from. Valid values for ```<xpos>``` and ```<ypos>``` depends on the Tile Map Mode. The ```<xoffset>``` and ```<yoffset>``` parameters can take a value between 0-7 and are used for per-pixel offset of the Tile Map.

For example, a ```<xpos>``` value of 10 will start drawing the layer from column 10 of the Tile Map. An ```<xoffset>``` of 4 would start drawing from the 4th pixel of column 10. This has the effect of "pulling" the image from right to left.

## Update Tile Layer

```VDU 23, 0, &C2, 28, 0```

This command will cause the Tile Engine to update the Tile Layer internally based on the scroll parameters but will not draw anything to the screen.

If the Tile Engine reaches the end of the Tile Map before completing the Tile Layer, the display will wrap to the start of the Tile Map.

## Draw Tile Layer

```VDU 23, 0, &C2, 29, 0```

This command will cause the Tile Engine to draw the Tile Layer to the display buffer. It will draw what is currently in the layer buffer.

This command is useful for drawing a static background to the screen as the contents of the layer do not need to be calculated.


## Update and Draw Tile Layer

```VDU 23, 0, &C2, 30, 0```

This command will cause the Tile Engine to update and draw the Tile Layer to the display buffer. It is a combination of the previous two commands that can be called with a single command.
