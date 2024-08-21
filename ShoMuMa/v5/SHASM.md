# SHASM

SHASM (SHomuma ASM) is a macro-assembly language for writing VNs more easily than manually.

## Preprocessor

`#pragma shellcode`

Enables shellcode in this visual novel.

`pragma autonewline`

Automatically creates newlines when running out of space in the dialog box.

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

Shellcode CPU

* `shellcode9` - use NDS9 to execute shellcode.  Faster and more capable than NDS7, but will block code execution like on GBA
* `shellcode7` - use NDS7 to execute shellcode.  Slower with less features, but runs in parallel with main engine

2nd screen behavior

[WIP]

### Vita




### Deck





### Windows Handheld





### Saturn

BG Renderer

* `bgs_on_vdp1` - render backgrounds on VDP1.  Allows for bypassing the limit of 5 backgrounds of VDP2, but slower and lacking in transparency support
* `bgs_on_vdp2` - render backgrounds on VDP2.  Fast and full transparency support, but limited to 5 BGs at a time



### Xbox 360





### Xbox One





### Xbox Series





### Windows





### macOS






## SHASM Instructions

### Scene Control

Instruction | Arguments | Description
|--|--|--|
`SCENE` | sceneName | Creates a new scene
`SCENEFLAGS` | flags | Defines the flags for the above scene.  Must always be specified even if flags do not change

### Graphics

Instruction | Arguments | Description
|--|--|--|
`BG` | bgIndex, path | Draw a background at the specified index using the image at specified path
`SPRITE` | spIndex, path | Defines a sprite at the specified index using the image at specified path
`SPRITEX` | spIndex, pos | Sets the X position of the upper right corner of sprite specified by index to value specified
`SPRITEY` | spIndex, pos | Sets the Y position of the upper right corner of sprite specified by index to value specified

### Text

Instruction | Arguments | Description
|--|--|--|
`SPEAKER` | text | Sets the speaker nameplate to the specified string
`TEXT` | text | Sets the scene text to the specified string

### Audio

Instruction | Arguments | Description
|--|--|--|
`AUDIO` | path | Plays a voice line with the specified path
`MUSIC` | path | Sets the scene BGM to the specified path

### Decisions

Instruction | Arguments | Description
|--|--|--|
`DECIDE` | text, path, scene | Create a decision with specified text, branching to specified scene if taken


### Code

Instruction | Arguments | Description
|--|--|--|
`CODEPTRS` | label | Assign code pointers to this scene.  If `SCENE_DECISIONS_BRANCHING_CODE` is set, acts as an array of pointers for each decision
`MOVL` | register, address | Load data into register from scratchpad
`MOVLI` | register, register2 | Load data into register from scratchpad, indirect addressing
`MOVS` | register, address | Store data from register to scratchpad
`MOVSI` | register, register2 | Store data from register to scratchpad, indirect addressing
`LDR` | register, var/const | Loads data from specified variable or constant into specified register
`STR` | register, var | Stores data from specified register to specified variable
`ADD` | register, register2/const | Add specified register to specified value
`SUB` | register, register2/const | Subtract specified value from specified register
`MUL` | register, register2/const | Multiply specified register by specified value
`DIV` | register, register2/const | Divide specified register by specified value
`CMP` | register, register2/const | Compare via subtraction specified register with specified value, setting flags as needed
`BEQ` | address | Branch to specified address if zero flag is set
`BNE` | address | Branch to specified address if zero flag is not set
`BMI` | address | Branch to specified address if negative flag is set
`BPL` | address | Branch to specified address if negative flag is not set
`BVS` | address | Branch to specified address if overflow flag is set
`BVC` | address | Branch to specified address if overflow flag is not set
`JSR` | address | Enter subroutine at specified address, automatically pushing previous PC to invisible stack
`RTS` | None | Return from subroutine, automatically popping PC from invisible stack
`JSC` | path, scene | Go to specified scene in file.smm specified by path, automatically ends code execution
`ENDCODE` | None | End execution and return to ShoMuMa scene processing

### Misc

Instruction | Arguments | Description
|--|--|--|
`EXIT` | None | Exit VN
`RESET` | None | Send virtual reset signal (reload .vn)
`SAVEGAME` | None | Save all variables to save data file
`LOADGAME` | None | Load all variables from save data file
`SHELLCODE` | path, bool | Execute shellcode, waiting until child process finishes if specified, requires unsafe be marked in .vn, processless platforms force wait

### Hardcoded Data

Line prefix | Arguments | Description
|--|--|--|
`.raw` | data | Insert raw data at this point
