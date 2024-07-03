# SHASM

SHASM (ShoMuma ASM) is a macro-assembly language for writing VNs more easily than manually.

## Preprocessor

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

### GBA

Sprites as

* `sprites_as_sprites` - Render sprites as HW sprites.  Allows max BGs and sprites, but many large sprites can result in underdraw (default)
* `sprites_as_bgs` - Render sprites as backgrounds.  Allows for larger sprites without underdraw but sprites + BGs limited to 4 total

Render Mode

* `tile_mode` - use GBA Mode 0.  Max HW backgrounds, and fast, but limited to 16 colors per tile and no precision drawing (default)
* `swapchain_mode` - use GBA Mode 4.  Unlimited SW backgrounds, 256 colors, precision drawing, but slow.



### DS

Sprites as

* `sprites_as_sprites` - Render sprites as HW sprites.  Allows max BGs and sprites, but many large sprites can result in underdraw (default)
* `sprites_as_bgs` - Render sprites as backgrounds.  Allows for larger sprites without underdraw but sprites + BGs limited to 4 total
* `sprites_as_3D` - Render sprites using DS 3D GPU.  Allows more color and more sprites on screen

Render Mode

* `tile_mode` - use DS Mode 0.  Max HW backgrounds, and fast, but limited to 16 colors per tile and no precision drawing (default)
* `swapchain_mode` - use DS 3D GPU.  Unlimited SW backgrounds, 262,144 colors, precision drawing

2nd screen behavior

[WIP]

### Vita




### Deck





### Windows Handheld





### Saturn





### Xbox 360





### Xbox One





### Xbox Series





### Windows





### macOS






## SHASM Instructions



SCENE

Defines a new scene with specified name.


SCENEFLAGS

Defines the flags for the scene.  Must ALWAYS follow SCENE, even if flags do not change.
