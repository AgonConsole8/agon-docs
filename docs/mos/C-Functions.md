# Using MOS C functions

MOS 3.0 added a new API, [mos_getfunction](./API.md#0x50-mos_getfunction), which allows programs to get the address of various C functions.  These functions comply with the Zilog eZ80 C calling conventions.

As MOS already provides an extensive API, the range of C functions this API provides is limited.  There are essentially three potential reasons why a function has been made available via this API:

1. The function cannot be made available via a traditional MOS API
2. The equivalent MOS API is difficult to use from programs written in C (in general because it uses the `IX(U)` register)
3. Direct access to the function may be useful for efficiency reasons

It is important to note that these functions are only accessible to programs written to run in the eZ80's ADL mode.  Z80 mode programs cannot use the `mos_getfunction` API, and would not be able to use these functions even if they could, as the calling convention is not compatible with Z80 mode code.

The complete list of functions available from the `mos_getfunction` API is provided in the [MOS C functions](#functions) section below.

## The C function calling convention

Zilog's documentation for their C function calling convention is described in a document with the very catchy name [ZDS II for eZ80Acclaim!: Calling C Functions from Assembly and Vice Versa](https://www.zilog.com/docs/appnotes/an0333.pdf).  This document is essentially an edited except from the "ZiLOG Developer Studio II eZ80Acclaim!® User Manual" with some additional examples.

When considering function pointers returned by the `mos_getfunction` API the notes around function naming conventions in Zilog's documentation are not really important.

If you are using these functions from a C program, assuming your compiler understands the Zilog eZ80 C calling convention (which is currently true for all C compilers for the Agon platform) you will need to let your compiler know the function prototype.

The underlying details of how the C function calling convention works should not be important to you if you are only using these functions from C.  You can skip ahead to the descriptions of the [functions available](#functions) from the `mos_getfunction` API.

However if you are using these functions from assembly then you will need to understand how the calling convention works.

### Passing parameters to functions

Function parameters are passed on the stack in reverse order, i.e. the rightmost parameter is pushed first, and the leftmost parameter is pushed last.  The leftmost argument is therefore at the top of the stack when the function is called.

After a function call, the caller is responsible for cleaning up the stack.  This means that the caller must remove the arguments from the stack after the function has returned.

Arguments of different types are always a multiple of 3 bytes in size, essentially because the eZ80's registers are 3 bytes wide in ADL mode.  As most variable types are not a multiple of 3 bytes in size, some bytes pushed to the stack will be ignored by functions that take arguments of those types.  C compilers that support the eZ80 C calling convention will automatically understand this and ignore any extra padding bytes on the stack when handling function arguments.  For example, a `char` type is 1 byte in size, and will be padded to 3 bytes when passed to a function.  The compiler will automatically ignore the extra 2 bytes.

The following table provides a guid the types of arguments that can be passed to functions, their size, and how they are arranged in memory on the stack.  As type names can vary, the table may show various common names used for the same underlying type, and is not exhaustive.  For example the table does not list `int8_t`, `unsigned char` or `BYTE`, as they are essentially equivalent to `char` and `uint8_t`.  The "memory" column shows how the values are arranged from the least significant byte to the most significant byte.  It is always a multiple of 3 bytes, with each individual value byte shown as `xx`; bytes that will be ignored by the function are shown as `??`.

| Type | Native Size | Size on stack | Memory (Low to high) |
|------|-------------|---------------|----------------------|
| `char`/`uint8_t` | 1 byte | 3 bytes | `xx ?? ??` |
| `short`/`uint16_t` | 2 bytes | 3 bytes | `xx xx ??` |
| `int`/`uint24_t` | 3 bytes | 3 bytes | `xx xx xx` |
| `long`/`uint32_t` | 4 bytes | 6 bytes | `xx xx xx xx ?? ??` |
| `long long`/`uint64_t` | 8 bytes | 9 bytes | `xx xx xx xx xx xx xx xx ??` |
| `float` | 4 bytes | 6 bytes | `xx xx xx xx ?? ??` |
| `double` | 4 bytes | 6 bytes | `xx xx xx xx ?? ??` |
| `void *` (any pointer) | 3 bytes | 3 bytes | `xx xx xx` |

* as noted above, the types in this table are not exhaustive, and similar types of the same length/size will be treated the same way
* `uint24_t` is not part of the C standard, but may be supported by some compilers that support the eZ80.  As the eZ80 includes several 24-bit registers when in ADL mode, this is also the default integer `int` type used by these compilers.
* the Zilog C compiler does not support the `long long`/`uint64_t` type, and this is not part of their C calling convention standard, but it is a common extension that other compilers for the eZ80 supports.  This type is not currently used by any of the functions available from the `mos_getfunction` API, but is included for completeness.
* owing to constraints of the eZ80 architecture, both `float` and `double` are IEEE-754 single precision 32-bit floating point values by compilers that target the eZ80.

### Calling C functions from assembly

As values are passed to functions using the stack, the process of calling a C function from assembly is generally to push arguments in reverse order.  As noted above, all arguments must be a multiple of 3 bytes in size; the nature of the eZ80 when running in ADL mode is that as you can only push wide registers to the stack you will always push 3-bytes to the stack.  This means that if you have an 8-bit value in an 8-bit register for an 8-bit argument (e.g. a `BYTE` or a `char`), you just need to ensure the value is in the least signficant register of a 3-byte register, and then push that register to the stack.  The other two bytes will be ignored by the function.

The `mos_getfunction` API returns a pointer to the requested function in the `HLU` register.  In principle your program needs to "call" this function, but the eZ80 does not have a version of the `CALL` instruction that can use a register as the address to call.  There is, however, a `JP (HL)` instruction.  There is more than one way to use this instruction to "call" a function, but the simplest is probably the following:

```Z80
; Call a function pointer in HL
; use `CALL call_HL` to call the function, ensuring return address is pushed
.call_HL	JP (HL)
```

With this in place, to call a function provided from the `mos_getfunction` API from eZ80 assembler code, you would do the following:

1. Push the arguments to the stack in reverse order
2. Ensure that the `HLU` register contains the address of the function to call
3. Perform a `CALL call_HL` instruction to push the return address onto the stack
4. Execution continues after that call, with the return value set as per the calling convention
5. Pop arguments off the stack

Step 2 here could be a call to `mos_getfunction` at that point in the process, or that could be done earlier in your program.  Please note however that there may in future be some [issues with caching function pointers](#modules) when [MOS Modules](./Modules.md) so please read the guidance below.

### Return values

The return value from a function will be placed in one or more of the eZ80's registers.

Exactly which registers are used depends on the type of the value returned.  The following table shows how different types of return values are returned.  As with the table above for arguments, the types in this table are not exhaustive, and similar types of the same length/size will be treated the same way.  Some types are returned across multiple registers; the table below illustrates register contents from the most significant byte to the least significant byte.

| Type | Return registers | Register contents |
|------|------------------|-------------------
| `char`/`uint8_t` | `A` | `xx` |
| `short`/`uint16_t` | `HLU` | `?? xx xx` |
| `int`/`uint24_t` | `HLU` | `xx xx xx` |
| `long`/`uint32_t` | `E:HLU` | `xx: xx xx xx` |
| `long long`/`uint64_t` | `BC:DEU:HLU` | `?? xx xx: xx xx xx: xx xx xx` |
| `float` | `E:HLU` | `xx: xx xx xx` |
| `double` | `E:HLU` | `xx: xx xx xx` |
| `void *` (any pointer) | `HLU` | `xx xx xx` |

* as noted above, the types in this table are not exhaustive, and similar types of the same length/size will be treated the same way
* as with the argument table, `uint24_t` is not part of the C standard, and the `int` type is a 24-bit value
* as with the argument table, `long long`/`uint64_t` is not part of the Zilog C calling convention standard, but is a common extension that other compilers for the eZ80 supports.  This type is not currently used by any of the functions available from the `mos_getfunction` API, but is included for completeness.

## Functions available from the `mos_getfunction` API {#functions}

The following functions are available from the [`mos_getfunction`](./API.md#0x50-mos_getfunction) API.

It should be noted that that whilst MOS APIs will return `FRESULT` or "status" values in the 8-bit `A` register, most of the underlying functions return an `FRESULT` or an `int` for their status, which are actually 24-bit values.  The calling convention means they will be returned in `HL(U)`.

This list is complete as of the MOS 3.0 release, but may change in future releases of MOS to include additional functions.

### 0x00 - `BYTE	SD_init();`

Initialises the low-level SD card handling system 

### 0x01 - `BYTE	SD_readBlocks(DWORD sector, BYTE *buf, WORD count);`

Read raw sector data from SD card.

`DWORD` is a 32-bit value, `WORD` is a 16-bit value, and `BYTE` is an 8-bit value.  The `buf` pointer is a pointer to a buffer of data that will be filled with the data read from the SD card.  The `count` parameter indicates how many bytes to read from the SD card.

### 0x02 - `BYTE	SD_writeBlocks(DWORD sector, BYTE *buf, WORD count);`

Write raw sector data to SD card.

This function works in a similar way to the `SD_readBlocks` function.  The `buf` pointer here is a pointer to the buffer of data used to write to the SD card.

### 0x03 - n/a

Calling `mos_getfunction` asking for this function number will succeed, but return a `NULL` pointer.

Reserved for potential future `SD_status` function

### 0x04 - n/a

Calling `mos_getfunction` asking for this function number will succeed, but return a `NULL` pointer.

Reserved for potential future `SD_ioctl` function

### 0x05 - `int	f_printf (FIL* fp, const TCHAR* str, ...);`

The FatFS `f_printf` function.  This function is documented in the [FatFS documentation](http://elm-chan.org/fsw/ff/doc/printf.html).

### 0x06 - `FRESULT	f_findfirst (DIR* dp, FILINFO* fno, const TCHAR* path, const TCHAR* pattern);`

The FatFS `f_findfirst` function, equivalent to the [`ffs_dfindfirst`](./API.md#0x94-ffs_dfindfirst) API call.  This function is documemented in the [FatFS documentation](http://elm-chan.org/fsw/ff/doc/findfirst.html).

Please note that for C functions, an `FRESULT` is an `int` type, and therefore will be returned in the `HLU` register.  This differs from the MOS APIs for FatFS which returns an 8-bit `FRESULT` value.  The return values however are otherwise identical.

### 0x07 - `FRESULT	f_findnext (DIR* dp, FILINFO* fno);`

The FatFS `f_findnext` function, equivalent to the [`ffd_dfindnext`](./API.md#0x95-ffs_dfindnext) API call.  This function is documemented in the [FatFS documentation](http://elm-chan.org/fsw/ff/doc/findnext.html).

As with the `f_findfirst` function, an `FRESULT` is an `int` type, and will be returned in the `HLU` register.

### 0x08 - `BYTE	open_UART1(UART * pUART);`

Equivalent to the [`mos_uopen`](./API.md#0x15-mos_uopen) API call.  Please refer to the API documentation for details of the `UART` structure.

### 0x09 - `int		setVarVal(char * name, void * value, char ** actualName, BYTE * type);`

The underlying function the [`mos_setvarval`](./API.md#0x30-mos_setvarval) API call uses.  Please see the API documentation for information about variable names and types.

Please note that the way this function works differs slightly from the API; the API handles creating pointers for some arguments, and getting the values from them to return in registers.

The parameters are as follows:
- `name` is the name of the variable to set.  As with the API, if this name is a pattern the first matched name will be set, unless an `actualName` pointer has been provided.
- `value` is a pointer to the value to be set, or if setting a number type variable this will be the actual number to set.
- `actualName` is a pointer to a pointer to a string.  Providing this pointer is optional, but if it is set then on first call the pointer it points to should be set to `NULL`.  After the function has been called, the pointer will be set to the actual name string of the variable that has been set.  Please note that it is important to not adjust that string in any way.  If you have used a pattern in the `name` parameter, and have provided an `actualName` pointer, then subsequent calls to this function will set the next variable that matches the pattern, and the `actualName` pointer will be updated accordingly.
- `type` is a pointer to a `BYTE` for the variable type to be set.  This is required.  This value will be updated after the function has been called to reflect the actual type of the variable that has been set; please note that varible types 3 and 4 ("expanded" and "literal string") will be replaced with either "string" or "number" as appropriate.

The return value is a [MOS status code](./API.md#status-codes), which will be in the `HLU` register.

### 0x0A - `int		readVarVal(char * namePattern, void * value, char ** actualName, int * length, BYTE * typeFlag);`

The underlying function the [`mos_readvarval`](./API.md#0x31-mos_readvarval) API call uses.

This function is very similar to the `setVarVal` function, and it's arguments work in a very similar way.  These are the differences:

- `value` can be `null` to measure the length of a variable, but otherwise must always point to a buffer into which the variable value will be read/copied.  This is in contrast to the `setVarVal` function which will accept a raw number as the value to be set for "number" type variables.
- a pointer to a 24-bit `length` value is required.  On entry, this indicates the length of the `value` buffer, if one has been provided.  If a matching variable is found, this value will always be updated to reflect the full length of the variable that has been read, or will be `0` if no variable was found.
- the `typeFlag` pointer is required.  On entry, if the value this points to is `3` then the variable being read will be "expanded" before it is returned.  This means that if the variable being read is a "macro" type, the returned string will be the expanded string with variables referenced in the macro replaced with their values.  If the variable being read is a "number", then it will be converted into a string.  On return, this value will be set to the underlying type of the variable that has been read, if a variable was found.

The return value from this function is a [MOS status code](./API.md#status-codes), which will be in the `HLU` register.  If a `value` buffer was provided, and the variable was found, then the value will be copied into that buffer.  If the buffer was not large enough to hold the variable then as much of the variable as will fit will be copied into the buffer, the `length` will reflect the total length of the variable (which will be more than was copied) and the function will return an "out of memory" status code.

### 0x0B - `int		gsTrans(char * source, char * dest, int destLen, int * read, BYTE flags);`

The underlying function the [`mos_gstrans`](./API.md#0x34-mos_gstrans) API call uses.

The "GSTrans" functionality in MOS is used to translate a source string that may include references to [system variables](./System-Variables.md) into a new string with those variables expanded.  It can also convert other special character sequences into non-printable characters.  Guidance on this can be found in the documentation for the [`echo`](./Star-Commands.md#echo) star command.

The arguments to this function are as follows:

- `source` is a pointer to the source "template" string to be translated
- `dest` is a pointer to the destination buffer where the translated string will be written.  If this is set to `NULL` then the function will not write to the buffer, but will still calculate the length of the translated string.
- `destLen` is the size of the destination buffer, if one is provided
- `read` is a pointer to an integer to store the number of translated characters read from source
- `flags` is a set of flags that control how the translation is performed.  These flags are described in the [`mos_gsinit`](./API.md#0x32-mos_gsinit) API documentation.

The `source` and `read` pointers are both always required.  The `dest` pointer is optional, but if provided it cannot be the same as the `source` pointer.

The GSTrans process essentially performs a call to the [`mos_gsinit`](./API.md#0x32-mos_gsinit) API and then calls [`mos_gsread`](./API.md#0x33-mos_gsread) until it reaches the end of the source string.

This function returns a [MOS status code](./API.md#status-codes), which will be in the `HLU` register.  Various return values are possible, and may indicate that the source string was not valid, or various other errors.

### 0x0C - `int	substituteArgs(char * template, char * args, char * dest, int length, BYTE flags);`

The underlying function the [`mos_substituteargs`](./API.md#0x35-mos_substituteargs) API call uses to perform [argument substitution](./Argument-Substitution.md).

This function works in a very similar manner to its corresponding API call.  For information on the arguments to this function, please see the API documentation.

The return value from this function is the total length of the substituted string.  If a destination buffer was provided that was not large enough to hold the entire substituted string then the function will fill the buffer with as much of the substituted string as will fit.

### 0x0D - `int	resolvePath(char * filepath, char * resolvedPath, int * length, BYTE * index, DIR * dir, BYTE flags);`

The underlying function the [`mos_resolvepath`](./API.md#0x38-mos_resolvepath) API call uses.

The `resolvePath` function resolves a file path.  The primary use for this function is to take a MOS-style file path that may include a [path prefix](./System-Variables.md#path-variables) or use other [system variables](./System-Variables.md) and to resolve it into a full path that can be used by the underlying file system, i.e. the [FatFS APIs](./API.md#fatfs-commands).  It can also be used to find files within a directory that match a given pattern.

The arguments to this function are as follows:

- `filepath` is a pointer to the file path string that is to be resolved
- `resolvedPath` is a pointer to the buffer where the resolved path will be written.  If this is set to `NULL` then the function will calculate the length of the resolved path.
- `length` is a pointer to an integer that on entry indicates the size of the `resolvedPath` buffer.  On exit, this will be set to the length of the resolved path.
- `index` is a pointer to a `BYTE` that is used to keep track of which version of a path prefix is being used.  This is used when resolving paths that use a prefix where the corresponding system variable contains multiple values.  If the `index` pointer is set to `NULL` then only the first matching path will be returned.  You can use this to manually iterate through the available prefixes.
- `dir` is a pointer to a `DIR` object.  This is optional, but if it is not provided then the function can only return the first matching path.  If a `DIR` object is provided, then repeated calls to this function can be used to find all matching paths.  This is useful for finding files that match a given pattern.
- `flags` is a set of flags that control how the path is resolved.  These flags are described in the [`mos_resolvepath`](./API.md#0x38-mos_resolvepath) API documentation.

This function will return a [MOS status code](./API.md#status-codes), in the `HLU` register.  For more guidance on possible return values, please see the API documentation for the [`mos_resolvepath`](./API.md#0x38-mos_resolvepath) API call.

### 0x0E - `int getDirectoryForPath(char * srcPath, char * dir, int * length, BYTE index);`

The underlying function the [`mos_getdirforpath`](./API.md#0x39-mos_getdirforpath) API call uses.

This function will return the directory for a given file path, omitting the leaf name.

The arguments to this function are as follows:

- `srcPath` is a pointer to the file path string that is to be resolved
- `dir` is a pointer to the buffer where the resolved path will be written.  If this is set to `NULL` then the function will calculate the length of the resolved path.
- `length` is a pointer to an integer that on entry indicates the size of the `dir` buffer.  On exit, this will be set to the length of the resolved path.
- `index` is used to determine which version of a path prefix is used

This function is purely a string manupulation function.  It does not perform any filing system operations, 

This function returns a [MOS status code](./API.md#status-codes), in the `HLU` register.  If the path contains a prefix that is not recognised, or if the prefix was recognised but the `index` value is out of range, then the function will return a "no path" status code.

### 0x0F - `int resolveRelativePath(char * path, char * resolved, int * length);`

This is the underlying function that [`mos_api_getabsolutepath`](./API.md#0x3c-mos_getabsolutepath) uses.

This function will take a relative path, i.e. one that may contain `.` or `..` path components, and resolve it into an absolute path.  Please note that this function does not perform a file path resolution, i.e. it will not resolve any path prefixes or system variables.  This function does however verify that the path provided is valid (at least in terms of does the containing directory exist), and will return an error if it is not.

At the time of writing, this function requires both the source `path` argument and a destination `resolved` pointer to be provided.  To come into line with other functions this may change in the future to allow for the `resolved` pointer to be `NULL` in order to calculate the length of the resolved path.  As with the other functions above, the `length` pointer is used to indicate the size of the destination buffer, and will be updated to reflect the length of the resolved path.

Returns a [MOS status code](./API.md#status-codes), in the `HLU` register.

### 0x10 - `void * getsysvars()`

Returns a pointer to the system variables area.  Directly equivalent to the [`mos_sysvars`](./API.md#0x08-mos_sysvars) API call.

The sysvars pointer will be returned in the `HLU` register.

### 0x11 - `void * getkbmap()`

Returns a pointer to the keyboard map.  Directly equivalent to the [`mos_getkbmap`](./API.md#0x1e-mos_getkbmap) API call.

The keyboard map pointer will be returned in the `HLU` register.


## A note for the future: MOS modules {#modules}

It is highly likely that a future version of MOS will introduce a [module system](./Modules.md) that will allow some parts of MOS to be loaded and unloaded at run-time.

What this means is that the address of a function returned by the `mos_getfunction` API might not be valid if the module that provides it has been unloaded.

Functions provided by "Core MOS" will always be present.  At this time the exact details of what will be included in "Core MOS" have yet to be finalised, but it is likely that all of the APIs and star commands present in MOS 2.3 will be included.  Whilst some new features added in MOS 3.0 are also likely to be included in "Core MOS" are also likely to be included, not all of them will be.

For now, the safest way to use the `mos_getfunction` API would be to call it each time you need to use a function, immediately before you call the function it returns.  This should guarantee that the module containing the function has been loaded and is present in memory.

