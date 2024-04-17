# VDU 23, 0, &C8: Context Management API

As of Console8 VDP 2.8.0 the VDP now has the ability to create and manage different graphical contexts.  This allows you to easily switch between different settings for drawing text and graphics on the VDP, and to save and restore these settings as needed.

Almost all settings related to the text and graphics system are stored in the context.  These include things like the currently selected fonts for the text and graphics cursors, text cursor behaviour, position and display settings, the active cursor (whether the context is in `VDU 4` or `VDU 5` mode), the selected graphics coordinate system and last graphics cursor positions, the currently selected font, viewports, GCOL painting mode, the selected bitmap, and so on.

The only graphical things that do _not_ change with the context are things that should be considered "global" in nature.  That includes the contents of buffers and graphical items that make use of buffers, such as bitmaps and font definitions.  Sprites are also "global" in nature.  Additionally data related to the current screen mode and the system palette are global across all contexts.

Changing the screen mode will reset the graphics system.  All saved contexts will be lost, and the current context stack cleared.

## Context concepts

There are two important concepts to understand when working with contexts, specifically the idea of a "context stack", and the idea of selecting a different context stack.

### Context Stacks

At any time you can choose to save the current graphics sytem state (context) to the context stack, or restore the last saved context from the stack.

This allows for a simple way to save the current graphics system state, make temporary changes, and then step back to that previous state.

This is a common concept in graphics programming APIs.  For example, in the HTML5 web canvas API, you can save the current state of the canvas, make changes, and then restore the previous state.  Similarly Apple's Core Graphics API has the same concept concept.

You will always have an active context, and a context stack.

### Selecting a Different Context Stack

The context management API also provides a way to select completely different context stacks.

This can be useful if you want to have completely different sets of settings for different areas of your program, or applying to drawing different parts of the screen.

## The API

The API to manage contexts is relatively simple, and consists of the following commands:

### `VDU 23, 0, &C8, 0, contextId`: Select context stack

This command will select a different context stack to use.

If there is no context stack with the given `contextId`, a copy of the current context stack will be created and saved to the ID, and the new context stack will be selected.

If you wish to explicitly save the current context stack to a `contextId` you should first delete the context stack and then use this command.

Once you have selected a context stack by ID, all subsequent changes to your context stack will be saved against that ID.

### `VDU 23, 0, &C8, 1, contextId`: Delete context stack

This command will delete the context stack with the given `contextId`.

Please note that deleting a stack from the store will not affect the current context stack, and will not affect the current context.

### `VDU 23, 0, &C8, 2, flags`: Reset

This command will reset the current context.  It accepts a bitfield of flags to indicate what aspects of the settings should be reset.  Sending a flags value of `0` will perform the same type of context reset that happens on a screen mode change.

The following flags are supported:

| Bit | Description |
| --- | ----------- |
| 0   | Reset graphics painting options |
| 1   | Reset graphics positioning settings, including graphics viewport and coordinate system |
| 2   | Reset text painting options |
| 3   | Reset text cursor visual settings, including text viewport |
| 4   | Reset text cursor behaviour |
| 5   | Reset currently selected fonts |
| 6   | Reset character to bitmap mappings |
| 7   | Reserved for future use |

These are broad reset options, resetting all settings in the given category to their default values.  More fine-grained control to reset individual settings are available via other VDU commands.

When all flags are set to `0`, this command performs a reset equivalent to changing the screen mode.  The text cursor behaviour and character to bitmap mappings will be left unchanged, as will the selected positioning system.

### `VDU 23, 0, &C8, 3`: Save context

This command will save a copy of the current graphics style state to the context stack so you can later revert any changes you make to it using a "restore".

### `VDU 23, 0, &C8, 4`: Restore context

This command will restore the last saved context from the context stack.

If there are no saved contexts on the stack, this command will do nothing.

### `VDU 23, 0, &C8, 5, contextId`: Save and select a copy of a context

Saves the current context to the context stack.  If there is a context stack saved with the given `contextId` then a copy of the most recent context at that `contextId` will be made and activated, replacing the current context.

If there is no context stack with the given `contextId`, then a copy of the current context will be saved to the stack, and you will be left with the current context.

Please note that this does _not_ change the current context stack.

### `VDU 23, 0, &C8, 6`: Restore all

This command is equivalent to calling "restore" repeatedly until there are no more saved contexts on the stack.  It reverts back to the first context saved on the current stack.

### `VDU 23, 0, &C8, 7`: Clear stack

This command will clear all saved contexts from the current context stack.  You will be left with the current context still active, but no history to "restore" to.

