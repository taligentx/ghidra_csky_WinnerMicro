# Ghidra Processor Module for C-SKY Winner Micro microcontrollers
This fork builds on the [original extension](https://github.com/leommxj/ghidra_csky/) for [Ghidra](https://github.com/NationalSecurityAgency/ghidra) to better support the Winner Micro W806 microcontroller and its variants:
  * Implementation of all 16 and 32-bit instructions compiled by the [wm-sdk-w806](https://github.com/IOsetting/wm-sdk-w806) 
    and [wm_iot_sdk](https://github.com/winnermicro/wm_iot_sdk/blob/master/README_EN.md) SDK examples.
  * Full implementation of single-precision floating point instructions
  * Full implementation of commonly seen DSP instructions + listing support for many additional instructions
  * Headers with data types and structs extracted from SDK libraries and processed for Ghidra's Data Type Manager
  * DWARF register mappings - potential for using Ghidra's debugger/emulator
  * Instruction bugfixes and updates

Example: With this module, Ghidra can now fully disassemble, decompile, and patch firmware for the [Zeeweii DSO3D12 oscilloscope](https://github.com/taligentx/ZeeTweak). Thanks to [@leommxj](https://github.com/leommxj) for the extensive work on the original extension, this variant would not be possible otherwise.

## Files
  * `C-SKY` - extension files that can be compiled using gradle. For normal usage, install the .zip from [Releases](https://github.com/taligentx/ghidra_csky_WinnerMicro/releases).
  * `C-SKY_ISA_Reference_Guides` - info on the ISA and the Winner Micro MCU, including dual translations (DeepL and Google) as needed.
  * `WM_SDK_Data_Types/wm-w806-sdk.gdt` - this file is for Ghidra's Data Type Manager to help identify SDK library items (data types, structs, functions, etc).
    * After importing a binary: CodeBrowser > Window > Data Type Manager > Top right "down arrow" menu > Open File Archive.

## Notes
  * See [FLSTweak](https://github.com/taligentx/FLSTweak) to parse Winner Micro .fls firmware files and extract bootloader and runtime images, as well as the starting memory address of each image.
    * Ghidra requires setting the address when initially importing an image for accurate analysis (File > Import File > Options > Base Address). For example, the Zeeweii firmware starts at `0x08010400`.
  * Similar to ARM, some Ghidra analyzer settings do not play well with C-SKY and can introduce errors or broken control flow:
    * Aggresive Instruction Finder: disable
    * Decompiler Parameter ID: disable
    * Non-Returning Functions - Discovered: disable
  * All C-SKY v2 binaries should be parsable, the added instructions are part of the C-SKY ISA and just happen to be utilized on the particular MCU I targeted.
  * If you come across firmware that has errors while disassembling, [post an Issue](https://github.com/taligentx/ghidra_csky_WinnerMicro/issues) - this module is not (yet) exhaustive, there are still many instructions that can be implemented for real-world use cases.

## Getting Started
1. Download the extension .zip file: [Releases](https://github.com/taligentx/ghidra_csky_WinnerMicro/releases)
    * Note that Ghidra extensions are built for specific Ghidra versions: https://github.com/NationalSecurityAgency/ghidra/releases
2. Start Ghidra > File > Install Extensions > Top right `+` icon > select the extension .zip file.
    * To update the extension, remove the existing extension, restart Ghidra, and install the updated extension.
3. After restarting Ghidra, add the target binary: File > Import File
4. Set Language: `C-SKY v2`
5. Select Options > Set base address - this is the memory address where the executable code starts.
    * For example, the W806 runtime starts at `0x08010400` by default.
6. Double-click the imported file to launch CodeBrowser and when prompted to analyze, select `No`.
7. Select Window > Memory Map and set permissions for the executable code portion of the binary to `RX` (disable write).
    * If you know that part of the binary is read-only data (embedded data, strings, images, etc), split that region and mark it as `R` (disable write and execute). Ghidra uses these parameters to detect code vs data.
8. Select Analysis > Auto Analyze. De-select `Non-Returning Functions - Discovered` to avoid spurious control flow issues.
9. Done! At this point, you should have a listing with at least some disassembled instructions, and then the fun starts!

## Release Notes
* 0.2
  - Support Ghidra 12.0
  - Add instructions (many)
  - Add C-SKY and Winner Micro ISA reference guides
  - Add Ghidra Data Type Manager archive for wm-sdk-w806 SDK libraries
  - Add DWARF register mappings for ELF import
  - Fixed instruction inversion bugs (especially lrw)
* 0.1.5 - initial fork release
  - Support Ghidra 11.3
  - Update for Xuantie XT804 (ck804ef) processor used in Winner Micro W806 MCU
  - Add instructions (with pcode)
    - 16-bit: tst16
    - 32-bit: tst32, bnezad32, ldm32, stm32, lrs.h, srs.h
    - DSP v2: ldbi.b, ldbi.h, ldbi.w, stbi.b, stbi.h, stbi.w
  - Add instructions (stub only, no pcode yet)
    - 16-bit: nie, nir, ipush, ipop
    - 32-bit: cprgr, cprcr, cpop, mul.u32, mul.s32, mula.32.l, ff1, mtcr, mfcr, psrclr, psrset, rte
    - DSP v2: add.64, addh.u32, sub.64, max.u32, max.s32, min.u32, min.s32, pldbi.d, ldbi.bs/hs, ldbir.b/h/w, pldbir.d, ldbir.bs/hs, stbir.b/h/w
    - FPU v2: single-precision instructions
