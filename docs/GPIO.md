# What is GPIO

GPIO, or General Purpose Input and Output, is typically an electrical interface on a computer that is not dedicated to a single task. On the Agon this is exposed as a series of pins for the user to interface to. This might be for input controls, such as a joystick or buttons, motion sensors, or outputs to control your Christmas tree lights, send data to other displays, microprocessors, or even connect to the internet.

Whilst sharing many similar pins, there are some slight variations across the range of Agon machines.

The first generation Agon Light has a double row of 32 pins, whereas the Agon Lght 2 has a double row of 34 pins. The additional pair of pins have been added for a battery power supply connection.

The Agon Console8 adds a few extra rows for additional connectivity to parts of the ESP processor.

The Agon Light 2 has an additional UEXT port. Whilst this replicates pins which are already on the main io connector bus, it is sometimes useful to have the duplicate pins for different purposes. For example, connecting a Nintendo Nunchuck controller on the UEXT, and a joystick on the main io bus.


## GPIO Pinouts - Original Agon Light

Viewed from the back, component side up, with the connector to the right of the board.

![](./images/iopinsAL1.png)

## GPIO Pinouts - Agon Light 2

Whilst the pin numbering system is different, the general layout is the same, apart form the last right hand two pins.

![](./images/iopinsAL2.png)

Note that the additional UEXT connector is mounted upside down to the normal numbering scheme, with connections as shown here.

![](./images/io_uext.png)

## GPIO Pinouts - Console 8

Viewed from the top front, component side up.

The Console 8's pin numbering is different and although they look similar at first, some of the pins do not align with those of the Agon Light, so do not plug an Agon Light peripheral into the io bus of a Console 8.
Pins marked * go via a transputer.

![](./images/iopinsC8.png)
