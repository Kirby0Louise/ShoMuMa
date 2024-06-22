
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

#### Scene Address Table

64-bit addresses into the .smm file for each scene.  If this .smm is the entry point, the first one is automatically loaded

#### Scenes

##### Standard Scene

SCENE_FLAGS

Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, SCENE_DIRECT_DATA_ENCODE, SCENE_CODE_ONLY, SCENE_HAS_DECISIONS, SCENE_IS_MENU)

* `SCENE_IS_MENU` - This scene is a menu.  Mutually exclusive with code/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_HAS_DECISIONS` - This scene has reader decisions.  Mutually exclusive with menu/code scenes and behavior is undefined if they are forcefully combined
* `SCENE_CODE_ONLY` - This scene is code only.  Mutually exclusive with menu/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_DIRECT_DATA_ENCODE` - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details

Background Table
* `SCENE_BG[X]` - Table of relative paths from .vn for each of the backgronds, drawn back to front.
* `GVAR_MAGIC` - End of background table.

Sprite Table
* `SPRITE_DATA[X]` - Relative paths from .vn for each of the sprites
* `SCENE_SPRITEX[X]` - 16-bit X position of sprite
* `SCENE_SPRITEY[X]` - 16-bit Y position of sprite
* `GVAR_MAGIC` - End of sprite table

Text Table
* `SCENE_SPEAKER` - Plaintext containing the text to be rendered in the speaker subtitle box for this scene
* `SCENE_TEXT` - Plaintext containing the text to be rendered in the subtitle box for this scene
* `GVAR_MAGIC` - End of text table

Audio Table
* `SCENE_BGM` - Relative path from game.vn to current scene BGM
* `SCENE_VOICE` - Relative path from game.vn to current scene voice
* `GVAR_MAGIC` - End of audio table

---

##### Decision Scene

SCENE_FLAGS

Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, SCENE_DIRECT_DATA_ENCODE, SCENE_CODE_ONLY, SCENE_HAS_DECISIONS, SCENE_IS_MENU)

* `SCENE_IS_MENU` - This scene is a menu.  Mutually exclusive with code/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_HAS_DECISIONS` - This scene has reader decisions.  Mutually exclusive with menu/code scenes and behavior is undefined if they are forcefully combined
* `SCENE_CODE_ONLY` - This scene is code only.  Mutually exclusive with menu/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_DIRECT_DATA_ENCODE` - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details

Background Table
* `SCENE_BG[X]` - Table of relative paths from .vn for each of the backgronds, drawn back to front.
* `GVAR_MAGIC` - End of background table.

Sprite Table
* `SPRITE_DATA[X]` - Relative paths from .vn for each of the sprites
* `SCENE_SPRITEX[X]` - 16-bit X position of sprite
* `SCENE_SPRITEY[X]` - 16-bit Y position of sprite
* `GVAR_MAGIC` - End of sprite table

Text Table
* `SCENE_SPEAKER` - Plaintext containing the text to be rendered in the speaker subtitle box for this scene
* `SCENE_TEXT` - Plaintext containing the text to be rendered in the subtitle box for this scene
* `GVAR_MAGIC` - End of text table

Audio Table
* `SCENE_BGM` - Relative path from game.vn to current scene BGM
* `SCENE_VOICE` - Relative path from game.vn to current scene voice
* `GVAR_MAGIC` - End of audio table

Decision Table
* `DECISION_TEXT[X]` - Text to be rendered in decision box
* `DECISION_FILE_BRANCH[X]` - Relative path to the .smm file to branch to if this option is taken
* `DECISION_SCENE_BRANCH[X]` - Offset into target .smm scene table to branch to

---

##### Code Scene

SCENE_FLAGS

Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, SCENE_DIRECT_DATA_ENCODE, SCENE_CODE_ONLY, SCENE_HAS_DECISIONS, SCENE_IS_MENU)

* `SCENE_IS_MENU` - This scene is a menu.  Mutually exclusive with code/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_HAS_DECISIONS` - This scene has reader decisions.  Mutually exclusive with menu/code scenes and behavior is undefined if they are forcefully combined
* `SCENE_CODE_ONLY` - This scene is code only.  Mutually exclusive with menu/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_DIRECT_DATA_ENCODE` - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details

---

##### Menu Scene

SCENE_FLAGS

Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, SCENE_DIRECT_DATA_ENCODE, SCENE_CODE_ONLY, SCENE_HAS_DECISIONS, SCENE_IS_MENU)

* `SCENE_IS_MENU` - This scene is a menu.  Mutually exclusive with code/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_HAS_DECISIONS` - This scene has reader decisions.  Mutually exclusive with menu/code scenes and behavior is undefined if they are forcefully combined
* `SCENE_CODE_ONLY` - This scene is code only.  Mutually exclusive with menu/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_DIRECT_DATA_ENCODE` - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details



## Instructions

Instruction | Opcode | Arguments | Description
|--|--|--|--|
`CMD` | 0xA0 | None | Indicates the following data is a command/instruction
`HIRAGANA` | 0xA1 | None | Indicates the following data is Hiragana, encoded as a single byte offset from the start of the unicode block.  Escape with 0xFF
`KATAKANA` | 0xA2 | None | Indicates the following data is Katakana, encoded as a single byte offset from the start of the unicode block.  Escape with 0xFF
`KANJI` | 0xA3 | None | Indicates the following data is Kanji, encoded as 2 byte offset from the start of the unicode block.  Escape with 0xFFFF
`UNICODE16` | 0xA4 | None | Indicates the following data is UTF-16.  Escape with 0xFFFF
`UNICODE32` | 0xA5 | None | Indicates the following data is UTF-32.  Escape with 0xFFFF
`LOADVAR` | 0xA6 | var_name, register | Load a variable from the save data into a register
`STOREVAR` | 0xA7 | var_name, register | Store a value from a register to a variable in the save file
`ENDCODE` | 0xFF | None | End of code block



## Special

Whenever plaintext is used, some commands are available, even if the scene is not marked as a command (0x00-0x9F are treated as ASCII).  These are mostly to facilitate typing other characters for foreign languages.

Commands that can be used in plaintext are:

* `CMD`
* `HIRAGANA`
* `KATAKANA`
* `KANJI`
* `UNICODE16`
* `UNICODE32`


