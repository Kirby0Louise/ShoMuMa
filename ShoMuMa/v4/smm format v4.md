
# ShoMuMa New Generation Visual Novel Format Specification

Revision - v4

Author - Kirby0Louise

## About

ShoMuMa is an assembly-like language for programming visual novels.  It can pack its data into very small and efficient files, and supports a variety of platforms.  SHASM is the accompanying macro-assembly language for more easily writing ShoMuMa visual novels.

This document covers all of the functions of the file format as well as instruction behavior, preprocessor directives, and more needed to fully utilize ShoMuMa and SHASM.

## Compiled Format

### [Visual Novel Name].vn

The .vn is the boot file of all ShoMuMa visual novels, and contains relevant header information, the path to the .smm entry point, and the version of ShoMuMa it was created with.

If you are creating an engine for ShoMuMa, you should filter out all files except .vn (or at least make filtering an option, such as with Windows OpenFileDialog filters).

| Address | Size | Variable Name | Description
|--|--|--|--|
0x0 | 0x9 | GVAR_MAGIC | ShoMuMa Magic ("MAGIC_SMM")
0x9 | Varies | MAIN_SMM | relative path to .smm file to launch at boot
0x9 + SIZE_OF(MAIN_SMM) | 0x2 | GVAR_SMM_VERSION | Revision of ShoMuMa
0xB + SIZE_OF(MAIN_SMM) | 0x40 | XTR_PLATFORM_COMPAT | Extra data, platform compatibility.  2-bits per platform, 0 = incompatible, 1 = Unstable/Low performance, 2 = Supported, 3 = Optimized.  Platform order is same as enum declaration for SHASM, starting at LSB

---


### [File].smm

Collection of scenes and menus.  Execution starts from the start of the file and continues linearly, unless a jump is specified.  Behavior for reading a scene beyond the end of the file is undefined!
You should probably implement a debugging trap for when this happens!

All scenes are either a regular scene, or a menu scene.

SCENE_FLAGS - Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, N/A, SCENE_HAS_DECISIONS, SCENE_DIRECT_DATA_ENCODE, SCENE_IS_MENU)

SCENE_IS_MENU - This scene is a menu and references different instructions
SCENE_DIRECT_DATA_ENCODE - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details
SCENE_HAS_DECISIONS - This scene has reader decisions.  Mutually exclusive with menu scenes and behavior is undefined if they are forcefully combined

---


## Instructions

Instruction | Opcode | Arguments | Description
|--|--|--|--|


---

## SHASM

SHASM (ShoMuma ASM) is a macro-assembly language for writing VNs more easily than manually.

### Preprocessor

`#pragma platform [PLATFORM]`

Defines the target platform.  This does not ensure that the VN only runs on said platform, but does help the assembler and engine optimize certain aspects.

* 0x0 - gba - GBA
* 0x1 - ds - DS(i)
* 0x2 - vita - Vita
* 0x3 - deck - Steam Deck
* 0x4 - winhand - Windows handheld (ROG Ally, Legion Go, etc)
* 0x5 - saturn - SEGA Saturn
* 0x6 - 360 - Xbox 360
* 0x7 - x1 - Xbox One
* 0x8 - xsx - Xbox Series
* 0x9 - windows - Windows
* 0xA - macos - macOS

`#pragma specific [SPECIFIC]`

Defines platform specific preprocessor info.  Platforms and associated info listed below.  Not including these uses the default behavior

#### GBA

Sprites as

* `sprites_as_sprites` - Render sprites as HW sprites.  Allows max BGs and sprites, but many large sprites can result in underdraw (default)
* `sprites_as_bgs` - Render sprites as backgrounds.  Allows for larger sprites without underdraw but sprites + BGs limited to 4 total

Render Mode

* `tile_mode` - use GBA Mode 0.  Max HW backgrounds, and fast, but limited to 16 colors per tile and no precision drawing (default)
* `swapchain_mode` - use GBA Mode 4.  Unlimited SW backgrounds, 256 colors, precision drawing, but slow.



#### DS

Sprites as

* `sprites_as_sprites` - Render sprites as HW sprites.  Allows max BGs and sprites, but many large sprites can result in underdraw (default)
* `sprites_as_bgs` - Render sprites as backgrounds.  Allows for larger sprites without underdraw but sprites + BGs limited to 4 total
* `sprites_as_3D` - Render sprites using DS 3D GPU.  Allows more color and more sprites on screen

Render Mode

* `tile_mode` - use DS Mode 0.  Max HW backgrounds, and fast, but limited to 16 colors per tile and no precision drawing (default)
* `swapchain_mode` - use DS 3D GPU.  Unlimited SW backgrounds, 262,144 colors, precision drawing

2nd screen behavior

[WIP]



#### Vita





#### Deck





#### Windows Handheld





#### Saturn





#### Xbox 360





#### Xbox One





#### Xbox Series





#### Windows





#### macOS





---
### SHASM Instructions

#### SCENE [SCENE_NAME]

Defines a new scene with specified name.


#### SCENEFLAGS [FLAGS]

Defines the flags for the scene.  Must ALWAYS follow SCENE, even if flags do not change.


