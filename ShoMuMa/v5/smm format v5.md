
# ShoMuMa New Generation Visual Novel Format Specification

Revision - v5

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
0xB + SIZE_OF(MAIN_SMM) | 0x1 | GVAR_HAS_SHELLCODE | Indicates if this VN uses `SHELLCODE`.  If disabled, the implementation MUST prevent execution of `SHELLCODE`.  If enabled, the implementation MUST warn the user before launch to only open unsafe VNs from sources they trust
0xC + SIZE_OF(MAIN_SMM) | 0x40 | XTR_PLATFORM_COMPAT | Extra data, platform compatibility.  2-bits per platform, 0 = incompatible, 1 = Unstable/Low performance, 2 = Supported, 3 = Optimized.  Platform order is same as enum declaration for SHASM, starting at LSB

---


### [File].smm

Collection of scenes and menus.  Execution starts from the start of the file and continues linearly, unless a jump is specified.  Behavior for reading a scene beyond the end of the file is undefined!
You should probably implement a debugging trap for when this happens!

#### Scene Address Table

64-bit addresses into the .smm file for each scene.  If this .smm is the entry point, the first one is automatically loaded

#### Scenes

A scene is the most basic building block of a ShoMuMa visual novel.  Scenes are composed of three parts, their header, data (optional, set SCENE_CODE_ONLY flag) and code.

##### Normal Scene

SCENE_FLAGS

Boolean values representing flags for this scene (N/A, N/A, N/A, N/A, SCENE_IS_MENU, SCENE_HAS_DECISIONS, SCENE_CODE_ONLY, SCENE_DIRECT_DATA_ENCODE)

* `SCENE_DIRECT_DATA_ENCODE` - Treats the data for the scene as a length and raw data encode for each of the parts (note that text is always encoded rather than pulled from a file), max 4GB for each data entry.  May speed up build times at the cost of huge bloat.  See directencode.txt for details
* `SCENE_CODE_ONLY` - This scene is code only.  Mutually exclusive with menu/decision scenes and behavior is undefined if they are forcefully combined
* `SCENE_HAS_DECISIONS` - This scene has reader decisions.  Mutually exclusive with menu/code scenes and behavior is undefined if they are forcefully combined
* `SCENE_IS_MENU` - This scene is a menu.  Mutually exclusive with code/decision scenes and behavior is undefined if they are forcefully combined

Background Table (Optional)
* `SCENE_BG[X]` - Table of relative paths from .vn for each of the backgronds, drawn back to front.
* `GVAR_MAGIC` - End of background table.

Sprite Table (Optional)
* `SPRITE_DATA[X]` - Relative paths from .vn for each of the sprites
* `SCENE_SPRITEX[X]` - 16-bit X position of sprite
* `SCENE_SPRITEY[X]` - 16-bit Y position of sprite
* `GVAR_MAGIC` - End of sprite table

Text Table (Optional)
* `SCENE_SPEAKER` - Plaintext containing the text to be rendered in the speaker subtitle box for this scene
* `SCENE_TEXT` - Plaintext containing the text to be rendered in the subtitle box for this scene
* `GVAR_MAGIC` - End of text table

Audio Table (Optional)
* `SCENE_BGM` - Relative path from game.vn to current scene BGM
* `SCENE_VOICE` - Relative path from game.vn to current scene voice
* `GVAR_MAGIC` - End of audio table

Decision Table (Optional)
* `DECISION_TEXT[X]` - Text to be rendered in decision box
* `DECISION_FILE_BRANCH[X]` - Relative path to the .smm file to branch to if this option is taken
* `DECISION_SCENE_BRANCH[X]` - Offset into target .smm scene table to branch to

Code
* `CODE` - Code for this scene to use.  See below for instructions.  Execution ends upon jumping to a new scene or hitting `ENDCODE`

##### Menu Scene

(Heavy WIP, and open to suggestions on how to improve)

Menu scenes have the same flags header as a normal scene, but follow a different format

Menu Table
* `MENU_TYPE` - Specifies the type of menu.  See below for values and their implementations
* `GVAR_MAGIC` - End of menu table

Menu Type | Value | Implementation
|--|--|--|
Main Menu | 0x00 | New Game, Continue, Options, Gallery, Exit
Options | 0x01 | Master Volume, BGM Volume, SFX Volume, Voice Volume, Audio Channels (Mono/Stereo/Surround on compatible systems), Text Speed, Reset
Save Data Management | 0x02 | Copy File, Erase File (optional on systems with limits on saves)
Save | 0x03 | Save to slot (Optional on systems with limits on saves)
Load | 0x04 | Load from slot (Optional on systems with limits on saves)
Checkpoint | 0x05 | Save, Load, Options, Return to Title, Next Chapter

Menu handling for implementations should be ideally called as a subroutine, and then returned from once VN processing should start again (i.e., don't `shomumaEngineMain(*main)` -> `menuHandler()` -> `shomumaEngineMain(*nextScene)` as this creates a stack memory leak, instead return the next scene to go to from `menuHandler()` and have `shomumaEngineMain()` handle where to jump to next using that return value).


---