# MOS Modules

As of MOS 3.0, MOS does not have a module system.  However it is highly likely that one will be introduced in the future, most likely in either in MOS 3.1 or MOS 3.2.

As we add new features to MOS, its size is growing, and we are fast running out of space in the flash memory of the eZ80 CPU where MOS is stored.  We therefore have a choice: stop adding new features, or find a way to allow those new features without taking up more space in the flash memory.

"Stop adding new features" wouldn't be a popular option as we have lots of ideas for new features we'd like to add to MOS.  One possibility is to work really hard to make MOS smaller, but that would have its limits too.

A better solution is to allow some parts of MOS to be loaded and unloaded from the SD card at run-time.  A module system.

!!! note

    It is possible that some of the new commands and APIs added in MOS 3.0 may be moved into modules in the future.  To guarantee that a program using these features will work in the future, you will need to ensure that your program is "module safe", and ensure your program has an [appropriate header](./Executables.md#advanced-header).

The MOS module system is still being designed and developed, and so the exact details of how it will work have not yet been finalised.  The following things howver are likely to be true.

## "Core MOS"

"Core MOS" will be the part of MOS that is always present, and which cannot be unloaded.  The exact details of what will be included in "Core MOS" are still being worked out, but for compatibility with existing programs written for MOS 1 and MOS 2, all of the star commands and APIs present up to and including MOS 2.3 will be included in "Core MOS".

Some new features added in MOS 3.0 will also be included in "Core MOS", but not all of them.  For example, the underlying functionality that supports [System Variables](./System-Variables.md) will need to be present in "Core MOS", however it may not be necessary to include all of the star commands and APIs used to manage them.  It is possible that some of these may be moved into a module.

As we add new features to MOS we will need to carefully consider which features need to be a part of "Core MOS" and therefore always present, and which can be implemented in modules.

## What MOS modules may contain

It is highly likely that MOS modules will be able to include implementations of API calls, star commands, as well as C functions.  The module design is likely to use an extensible format to allow for future expansion, allowing us to add new types of functionality to modules in the future.

## How modules are likely to work

MOS modules will almost certainly be loaded into the ["moslet" area of memory](../MOS.md#memory-map).  This area of memory is defined as the RAM from address `0xB0000` to `0xB7FFF`, which is 32 kilobytes in size, which will therefore be the module size limit.

When attempting to use a star command or API call that is implemented in a module, MOS will take care of loading the module into memory as required.  (This will be fairly sophisticated behaviour so as to support calling features in one module present in another module, and so on.)

The reality of the Agon platform however is that access to RAM from the eZ80 CPU is not, and cannot be, restricted.  A program can therefore potentially use the moslet area of memory should it choose to do so.

Existing programs may therefore be using the moslet area of memory.  There could therefore be problems if a module is loaded there, as the program's code or data could be overwritten, causing it to crash or behave unexpectedly.

The solution to this will be for programs to be "module safe".

A new [advanced executable header](./Executables.md#advanced-header) format has been introduced with MOS 3.0 to allow for programs to indicate to MOS whether they are "module safe" or "module compatible".

## "Module safe" programs

A program is "module safe" if it does not use the moslet area of memory.

This means that the program can safely use any and all of the features provided by modules, and will not be affected by a module being loaded into RAM.

## "Module compatible" programs

A "module compatible" program is one that uses the moslet area of memory, but does so in a way that is compatible with modules.

When MOS detects a program that is marked as "module compatible", and the program attempts to use a feature that is implemented in a module, it will save the contents of the moslet area of memory to disk before loading the module.  When the module has finished executing, MOS will restore the contents of the moslet area of memory from disk.

A "module compatible" program must ensure that any API calls it makes will not use the moslet area of memory for data buffers.  Ideally API calls implemented in modules will detect when they are being given a buffer that overlaps the moslet area and return an error, but this cannot be guaranteed.

It will be safe for a "module compatible" program to use "Core MOS" APIs to be passed buffers that overlap the moslet area of memory.

## Using modules from moslets

As moslets are designed to be loaded into the same area of memory as modules, it is not possible to use features provided by modules from moslets, and unlikely that this will be possible in the future.

It should be noted however that the purpose of moslets was essentially to provide a way to add new star commands to MOS.  As modules provide a different way to add new star commands to MOS, arguably we do not need to add support for using modules from moslets.

Unfortunately this means that if you write a moslet today that uses a MOS 3.0 API that is later moved into a module, it will not be able to run in future versions of MOS.  Calls to that API will fail with an error indicating that the API is not available.  You will likely need to rewrite your moslet to be a module instead that provides a star command.

## The future of modules

The exact details of the module system will likely change and evolve as the system is developed.

Some possibilities for the future include:

* "Driver" modules
    * One of the oldest ideas we had for a MOS module was for a "joystick module".  The concept was to define a standard set of Joystick APIs, but to load a joystick driver module specific for the joystick interface being used that implemented those APIs.  Whilst the Agon community has _mostly_ settled on the joystick interface system used by the Agon Console8, other interfaces do exist.
    * Other types of "driver" modules could be considered
* I/O modules
    * A concept being considered for a future version of MOS is "unified I/O", which would allow lots of different types of I/O to be performed in a consistent way
    * An I/O module could contain data that identifies which "stream" identifier it implements connections for, and implement a set of functions to support that stream
* User-defined/provided modules
    * In principle, users should be able to create their own modules to extend MOS
    * These could add new star commands, APIs, or C functions, or whatever other functionality is supported by the module system
    * In the case of APIs, we may need to have a registration system to ensure that the same API number does not get multiple different and incompatible/conflicting implementations
* Non-code modules
    * If modules are implemented in an extensible format, this may allow for modules to be used to store other things, such as data files, bitmaps, or other resources
* User-space modules
    * It may be possible to allow the module system to be used by programs to load and unload modules into their own address space, rather than the moslet area of memory
    * By using user-space modules, a program could potentially load/unload multiple modules simultaneously, and use them to extend the program's functionality
    * The use of non-code modules could be useful here; for example a game could use modules for level data, or other resources

## More information

At the time of writing the full details of the MOS module system are still being worked out.  If you wish to learn more and see how these ideas develop you can check the corresponding [issue on GitHub](https://github.com/AgonConsole8/agon-mos/issues/2).

