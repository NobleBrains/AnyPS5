# libSceUsbStorage

## change [the host passthrough](../../../core/libs/prx/libSceUsbStorage/src/HostVolumes.cpp)

- [the host passthrough](../../../core/libs/prx/libSceUsbStorage/src/HostVolumes.cpp) (libSceUsbStorage) - scans devices only with `APS5_USB_STORAGE_PASSTHROUGH` set; `ANYPS5_STORAGE_FOLDER` mounts either way (not a real USB mass-storage device). Devices are host-mounted USB FAT/exFAT volumes: on Linux, whole-FS `vfat`/`msdos`/`exfat` entries in `/proc/self/mountinfo` under a USB class 08 sysfs interface; on [Windows](../../../core/libs/prx/libSceUsbStorage/src/HostVolumesWindows.cpp), the first drive letter per FAT/FAT32/exFAT volume reporting `BusTypeUsb`; none on macOS. NTFS and FUSE exFAT (`fuseblk`) are excluded. `sceUsbStorageInit` scans and a thread rescans at 1 Hz; volumes keep their ID (lowest free of 8 slots) while mounted, with `/mnt/usb<id>` resolving to their root until dropped or `sceUsbStorageTerm`. Console numbering is unknown. Closes when: measured on a PS5

## change [the map requests](../../../core/libs/prx/libSceUsbStorage/Export.cpp)

- [the map requests](../../../core/libs/prx/libSceUsbStorage/Export.cpp) (libSceUsbStorage) - `sceUsbStorageRequestMap` and `sceUsbStorageRequestMapWSB` follow [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/f68923fe807f657348e6719ce658bcbd0a2b1c72/src/core/libraries/usbd/usbstorage.cpp#L245-L279): mount point is `/mnt/usb<id>` plus the directory (root if null/empty), last arg receives 0, and null outputs are skipped. Throws (answers unknown) on missing directory, `.`/`..` or root-name (Windows drive) components, backslash, mount point >127 bytes, duplicate host dir (including case-insensitive or symlink aliases), nonzero flags/volume size, or nonzero 4th `RequestMapWSB` arg. Unlike shadPS4, `/mnt/usb...` directories are not returned unchanged. SharpProspero's mapping limit (0x80f40015) is not enforced. Closes when: measured on a PS5

# libSceVdecsw

## change [libSceVdecsw](../../../core/libs/prx/libSceVdecsw/Export.cpp)

- [libSceVdecsw](../../../core/libs/prx/libSceVdecsw/Export.cpp) (libSceVdecsw) - layouts follow shadPS4 `vdecsw.h`, checked vs PPSA05684 (`movie-decoder-vdecsw.cpp`: asserts `OutputInfo+0xb` `pictureCount == 1`, reads `InputResult+0x10` frame count). Compute, memory, decode input structs and first 0x28 of 0x50B config match libSceVideodec2, which decodes. 0x78B AVC pic info fills only Videodec2 leading + cropping fields; 0xb8B HEVC (`vdecsw.h`) fills timestamps, coded size, bit depth, IDR/IRAP flags, profile, level and conformance window (chroma samples, copied to frame cropping fields, unit unknown). OUTPUT_PENDING (0x81510115), INPUT_QUEUE_EMPTY (0x81510116), DECODE_PENDING (0x81510117) come from asserts. Queues <=4 pics, holding last until `sceVdecswFinalizeDecodeSequence` (console depth unknown). Sync calls retry TrySync (yielding only), so another thread must drain outputs.

# libSceVideoOut

## change [sceVideoOutVrrPegToFixedRate, sceVideoOutVrrUnpegFromFixedRate]

- [sceVideoOutVrrPegToFixedRate, sceVideoOutVrrUnpegFromFixedRate](../../../core/libs/prx/libSceVideoOut/src/Output.cpp) (libSceVideoOut) - PPSA05684 passes the handle (plus two zero args to `PegToFixedRate`, meaning unknown) and ignores results. VRR is unmodelled (always fixed rate), so both only validate the handle and return 0. Checked direct calls pass 0 for args 2-3 (tail-jump callers unknown). A PS5 (FW 11.60 payload) returned 0x8029000b for handle -1 in both calls (returned here before arg checks); closed/non-(-1) invalid handles and non-zero args are unmeasured and throw. On valid handles that payload returned 0x8029001c in 7/7 tests (3 printed "System processes should use sceVideoOutSysSetDrrPeg() instead"), so title-process results remain unmeasured/unmodelled.

## change [sceVideoOutIsOutputSupported]

- [sceVideoOutIsOutputSupported](../../../core/libs/prx/libSceVideoOut/src/Output.cpp) (libSceVideoOut) - mode 1 59.94 Hz; mode 15 119.88 Hz needs >=120 Hz SDL display (else 0, `ConfigureOutput(15)`=`UNAVAILABLE_OUTPUT_MODE` 0x80290019, [PS5_Vk](https://github.com/mihawk-99/PS5_Vulkan/blob/f410e67a0241b3e8edcdbcd2a9b285dfc4a412ed/docs/HARDWARE_FINDINGS.md#L3793-L3835) 1080p/1440p) & attr3 bit 0x40 (checked 1st, order unmeasured; 0x80040 [PS5_Vk](https://github.com/mihawk-99/PS5_Vulkan/blob/5b5e4fc2d80fdb67a4f61ba9e6a8a424026bb8a5/docs/HARDWARE_FINDINGS.md)). Slower display->mode 1+event (#7877, unmeasured); no VRR/mode-switch. Modes 4,7-8,12-14,16-17,19->`VIDEO_OUT_ERROR_UNSUPPORTED_OUTPUT_MODE`; mode 18 & PPSA12544 `0xd000000a` (3840px)->0x8029001E (`VIDEO_OUT_ERROR_UNKNOWN_OUTPUT_MODE`, ours; FW 13.40 2026-10-19 & 11.60 2026-11-09 opts=NULL match; other PS5: 0x80290016). Opts w3 OK; PPSA26344 `0xCD0946`: `(h,0xf,0,0,0)==1`.

## change [sceVideoOutSysGetBus, sceVideoOutSysAddSetModeEvent2, sceVideoOutGetPipelineStatus]

- [sceVideoOutSysGetBus, sceVideoOutSysAddSetModeEvent2, sceVideoOutGetPipelineStatus](../../../core/libs/prx/libSceVideoOut/src/Event.cpp) (libSceVideoOut) - FW 4.03 `SysGetBus(h)` reads slot `busType` for handles 1-6 (`0x8029000b` if absent/closed); here `h` is from `Open`. `SysAddSetModeEvent2(eq, idx, udata)` needs `idx` 0/1 (`0x80290001` else) and registers event `0x73` for `idx`; unlike FW, `registerVideoOutEvent` needs port `idx+1` open (else `VIDEO_OUT_ERROR_INVALID_HANDLE`). `GetPipelineStatus(out, rsv)` needs non-null 32B `out` & `rsv==0` (`0x80290002` else), mapping `ioctl 0x80208218` 0-4 via `{0,0,0,1,0}` to `out[0]` (rest 0); no host device: returns `0x802900FF` untouched (like `DriverGetLatestEopHandle`). PPSA31544 `libSceAgcDriver.sprx` calls `SysGetBus` w/ `GetBufferLabelAddress`/`SubmitEopFlip` handle; `libScePsml.sprx` calls `SysAddSetModeEvent2(eq,0,udata)` (ret ignored) & `GetPipelineStatus(&buf,0)` (`out[0]==1`).

## change [sceVideoOutOpen, bus 0x20]

- [sceVideoOutOpen, bus 0x20](../../../core/libs/prx/libSceVideoOut/src/Output.cpp) (libSceVideoOut) - PPSA05666 calls `sceVideoOutOpen(255, 0x20, 0, NULL)` (`SCE_VIDEO_OUT_BUS_TYPE_HMD2_MAIN`) after `sceHmd2ReprojectionEnableVrMode(0x2000F)`. Non-game PS5 processes return `0x80290001` regardless of headset state, user 255/0 or null/zero param (returned before other checks; game+headset unmeasured). Unmodelled (needs VR backend): it passes that handle to `RegisterBuffers2` (set 0, 2 bufs, 4000x2040, tiling 0, pitch 0, fmt `0x8100000022000000`, cat 16, null opt; `data` -> 16B `{ptr, 2}` from `sceHmd2ReprojectionEnableMirroring` holding 2000x2040 per-eye texture descriptors), `ConfigureOutput`/`IsOutputSupported(0x2000F)` and GPU buffer-0 wait; passing `0x80290001` to `RegisterBuffers2` throws (console unmeasured). TV bus returns `0x8029001E` for HMD modes `0x20001`, `0x2000C`-`0x20010` on PS5 and here.

## change [Bluetooth DualSense haptics]

- [Bluetooth DualSense haptics](../../../core/libs/prx/libSceVideoOut/src/PadInput.cpp) (libSceVideoOut) - controller queue depth and console filter unmeasured; motor/haptics interplay measured on Windows 11 (Windows only). Vibration ports average to 3 kHz in 16-frame boxes (not console low-pass) and send report 0x32 as 32-frame blocks (once 2 buffered, >=10.67 ms apart, audio-clocked), requesting 64-frame (21 ms) controller queue (16 dropped ~1/s); host buffer holds newest 160 frames (53 ms). `scePadSetVibration` sends SDL report 0x31 (haptics off), resending haptics 0x31 100 ms after rumble stops; failed writes mute ports (keeping triggers, no rumble) until reconnect. With `openHaptics`, rumble feels weaker/smoother than skipped (skipped matches USB & Astro's Playroom PS5 rumble, which uses haptics audio). Closes when: clock drift and rumble interplay are measured on a title using both.

# libSceVideoStreamingEngine

## change [sceVideoStreamingEngineDestroyMediaKeySystemAccess

- [sceVideoStreamingEngineDestroyMediaKeySystemAccess, sceVideoStreamingEngineMediaKeySessionGenerateRequest, sceVideoStreamingEngineMediaKeySessionGetExpiration, sceVideoStreamingEngineMediaKeySessionGetKeyStatuses, sceVideoStreamingEngineMediaKeySessionGetSessionId, sceVideoStreamingEngineMediaKeySessionLoad, sceVideoStreamingEngineMediaKeySessionSetEventHandler, sceVideoStreamingEngineMediaKeySystemAccessCreateMediaKeys, sceVideoStreamingEngineMediaKeySystemAccessDestroyMediaKeys, sceVideoStreamingEngineMediaKeySystemAccessGetKeySystem, sceVideoStreamingEngineMediaKeySystemAccessSetCdmStorageConfig and sceVideoStreamingEngineMediaKeysCreateSession](../../../core/libs/prx/libSceVideoStreamingEngine/Export.cpp) (libSceVideoStreamingEngine) - names/NIDs in [PS5 3.20](https://github.com/DNNDHH/PS5-3.20_Libs/blob/b2a77c09cd71c01f34c790cfbe98c751a13262ad/libSceVideoStreamingEngine.c); no prototypes, 0-arg to resolve imports (not ABI, don't call).

# libSceVideodec

## change [sceVideodecCreateDecoder, sceVideodecDecode, sceVideodecDeleteDecoder, sceVideodecFlush, sceVideodecQueryResourceInfo and sceVideodecReset]

- [sceVideodecCreateDecoder, sceVideodecDecode, sceVideodecDeleteDecoder, sceVideodecFlush, sceVideodecQueryResourceInfo and sceVideodecReset](../../../core/libs/prx/libSceVideodec/Export.cpp) (libSceVideodec) - struct layouts and error codes follow [shadPS4's header](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/videodec/videodec.h) and [implementation](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/videodec/videodec.cpp), unmeasured on PS5 (console structs and rejected-arg errors unknown). Runs via libSceVideodec2 (FFmpeg H.264), copying frames with 64-byte aligned pitch and coded height. Null/struct-size checks in `DeleteDecoder`/`Reset`, handle check in `Reset`, and pic-info size check in `Decode` are extra (not in reference or PS5-measured).

# libSceVisionManager

## change [sceVisionManagerGetLibraryVersion, sceVisionManagerGetNumOfRegisterUserDataSlot

- [sceVisionManagerGetLibraryVersion, sceVisionManagerGetNumOfRegisterUserDataSlot, sceVisionManagerGetWorkingMemorySize, sceVisionManagerRequestFindPad, sceVisionManagerRequestFindPadCorrespondRegion, sceVisionManagerRequestPadTracking, sceVisionManagerGetResultOfFindPad, sceVisionManagerGetResultOfFindPadCorrespondRegion and `-R5dI6KjYpY`](../../../core/libs/prx/libSceVisionManager/Export.cpp) - layouts unknown; stateless, per FW 13.42 sprx (disasm+x86-64, not PS5; reg types). Err 0x80E70000: `LibVer` writes 0x1020000214072421; `UserDataSlot(v,c)` writes 16 (0x1017000014043000) or 30 (0x1020000214072421); `WorkMem` writes 0, 0x1600000, 0x4200000 (unchecked). Pad fails (0x80E70004 if valid): `FindPad` needs ptrs & `0<*p<5`(signed); `Correspond` adds `region` 1-4 & `result`; `Tracking` ignores arg1, limit 4; `GetResult*` need 3 ptrs. `-R5dI6KjYpY(i,o)` zeros 32B `o`, `o[0]`=5 if `max(i[0],i[1])`(u) 10-29 else 10 if >=30.

# libSceVoiceChat

## change [sceVoiceChatAbortRequest, sceVoiceChatRegisterChannelEventHandler

- [sceVoiceChatAbortRequest and 25 event/handler exports](../../../core/libs/prx/libSceVoiceChat/Export.cpp) (libSceVoiceChat) - Register{Channel,Daemon,Member,Voice}EventHandler, RegisterPartyUserStateChangedHandler, {Start,Stop}{Accessibility,Channel,Daemon,Member,RangedMember,Voice}Event, Unregister{Accessibility,Channel,Daemon,Member,Mic,RangedMember,Voice}EventHandler, and UnregisterPartyUserStateChangedHandler return 0 offline without a PSN voice chat daemon; listeners and audio capture events proceed without throwing.

# libSceVoiceQoS

## change [sceVoiceQoSGetRemoteEndpointAttribute]

- [sceVoiceQoSGetRemoteEndpointAttribute](../../../core/libs/prx/libSceVoiceQoS/Export.cpp): FW 11.00 (`3vmrvQ4rvgs#D#A`, VA 0x1960, 171B) and 13.40 (VA 0x1720, 185B) check init, non-null out, remote id >= 0 and 7/15 slot capacity (`appType & 0x10080000`), then under lock accept only selector 7 (1B) and read slot+0's port mute flag (`sceVoiceGetMuteFlag`, 13.40 GOT `0x80d0`). 13.40 `sceVoiceQoSCreateRemoteEndpoint` creates that port via `sceVoiceCreatePort` at slot+0; 13.40 `libSceVoice.sprx` `sceVoiceGetMuteFlag` (`Pc4z1QjForU`, VA 0x18600) returns `port.muted || mutedAll` (false initially, so unconnected endpoints read unmuted). Without a created endpoint, invalid-arg is returned rather than libSceVoice port 0 (matching local getter). The output voice port is unmodelled (see create entry).

# libc

## change [sigvec](../../../core/libs/prx/libc/src/LegacySignals.cpp)

- [sigvec](../../../core/libs/prx/libc/src/LegacySignals.cpp) (libc, libSceLibcInternal) - unmeasured on PS5/macOS; Windows fixture pins modules until exit (DLL unload crash cause unknown). Matches FW 13.42 `libSceLibcInternal` `yhxKO9LYc8A` emulated with `_sigaction`/`sigemptyset`; ABI, low-word mask & restart/intr inversion follow FreeBSD 11.4 [`sigcompat.c`](https://github.com/freebsd/freebsd-src/blob/8c639945a44588f6cfdddfb057bb5aad88735da0/lib/libc/compat-43/sigcompat.c) and [`signal.h`](https://github.com/freebsd/freebsd-src/blob/8c639945a44588f6cfdddfb057bb5aad88735da0/sys/sys/signal.h). Uses guest `sigaction` (absent throws). Inherits [guest signal gaps](libkernel.md): restart, `SA_ONSTACK`, `SA_SIGINFO` unapplied; mask, `SA_NODEFER`, `SA_RESETHAND` applied. Clears upper mask words; queries return low word. Guards old-vector write (`EFAULT` leaves unchanged); other invalid memory is UB.

## change [scriptingGetMem, scriptingFreeMem]

- [scriptingGetMem, scriptingFreeMem](../../../core/libs/prx/libc/src/ApplicationHeap.cpp) (libc) - Unity allocator hooks, unmeasured on PS5 (exporting console library unknown). `(alignment, size)` signature, min alignment 16, and null on non-pow2 follow [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/df5b9b92be7dfd463b47756b707d6d356a9f0550/src/libs/libKernel.cpp); 1-arg free follows [Raeen's contract](https://github.com/Whoisraeen/Raeen/blob/2db56923204b89719916b95e29c2f3c4e1d9f90b/docs/unity-dlsym-blocker.md). Uses libc's guest heap, not the title's allocator (PPSA18194 Metal Gear Solid Master Collection ships `libc.prx` already initializing it; calling it again would re-init; its `Il2cppUserAssemblies.prx` looks up `scriptingGetMem` by name for 4 KiB-aligned 256 KiB blocks). OOM/overflow return null.

## change [%s in the wide printf family]

- [%s in the wide printf family](../../../core/libs/prx/libc/src/FormattingWide.cpp) (libc) decodes narrow strings like title `libc.prx` (<=3-byte UTF-8, allowing overlongs/surrogates, NUL-terminated; checked side-by-side vs PPSA07923, not on PS5). Invalid leads, 4-byte leads or truncated chars make `swprintf`/`vswprintf` return -1 with `errno` `EILSEQ` (86), keeping prior output; precision caps decoded chars after always decoding the first, and empty results skip width padding. `%c` uses the low byte and fails >=0x80, setting `EILSEQ` only for 0x80-0xBF and 0xF0-0xFF (not `EOF`, like `btowc`). `wprintf`/`vfwprintf` fail with `EILSEQ` without writing, and secure/streamed forms' error output is unverified on console.

## change [acosh](../../../core/libs/prx/libc/src/Math.cpp)

- [acosh](../../../core/libs/prx/libc/src/Math.cpp) (libc) - adapted from FreeBSD msun [`e_acosh.c`](https://github.com/freebsd/freebsd-src/blob/release/11.4.0/lib/msun/src/e_acosh.c). For x > 2 it calls host `log`, so 110/100,000 random args differ by 1 ulp vs PPSA07923/PPSA05916 `libc.prx` (0 with msun `log` #10245); 1 < x <= 2 (`log1p`), x=1 and domain errors match bit-for-bit. x < 1 and negative-sign NaNs raise `FE_INVALID` and `errno` `EDOM` (msun sets none); positive NaNs return quieted. Matches PS5 (FW 11.60 payload, `libSceLibcInternal`) bits, `errno` and flags across 23 double / 22 float inputs (both-sign/kind NaNs, +-Inf, +-0, min subnormal..1e308, incl. negative NaN); only 5 finite args >= 2 tested, and title processes / FW 13.x are unmeasured.

## change [fgetws, getwc, getwchar, putwc, putwchar]

- [fgetws, getwc, getwchar, putwc, putwchar](../../../core/libs/prx/libc/src/FormattingWide.cpp) (libc) - UTF-8 wide stream I/O; `fgetws` follows title `libc.prx` disassembly (PPSA13934, PPSA07923, PPSA05916, not console-measured): size < 2 returns null untouched; stores low 16 bits of `fgetwc` until 32-bit `\n`, `-1` or full, returning null-terminated buffer if any char stored (even after read error). Null buffer/stream sets `errno` 22 (console dereferences). Pre/post-read and flag-byte-1 `0xC` calls and wide buffering are unmodeled/unverified. FW 11.60 payload system libc failed `F0 9F 98 80` (`EILSEQ` + 3 subsequent -1s) and read `C0 80\n` as `[0]`+EOF; unapplied as title decoding is unmeasured. `fgetwc` splits >BMP as UTF-16 surrogate pairs (buffered until seek), rejects overlongs, >U+10FFFF and `F8`-`FF` (`EILSEQ`), and round-trips 3-byte surrogates from `putwc`.

## change [ftrylockfile]

- [ftrylockfile](../../../core/libs/prx/libc/src/RuntimeSupport.cpp) (libc) - tries the host CRT stream lock used by `flockfile`/`funlockfile` and stdio calls: returns -1 if held by another thread, 0 if acquired/recursively owned (released per `funlockfile`), like FreeBSD 11 [`_ftrylockfile`](https://github.com/freebsd/freebsd-src/blob/8c639945a44588f6cfdddfb057bb5aad88735da0/lib/libc/stdio/_flock_stub.c#L89-L104). Exports hold the lock only per host call, not across calls or guest `FILE` prefix updates. Uses glibc `ftrylockfile` on Linux; on Windows (UCRT lacks try-lock), calls `TryEnterCriticalSection` on `_lock_file`'s `CRITICAL_SECTION` via private `__crt_stdio_stream_data` (`ucrt/inc/corecrt_internal_stdio.h`, SDK 10.0.19041-10.0.26100; fragile across CRT changes). Null/closed streams throw (FreeBSD dereferences).

## change [sincos](../../../core/libs/prx/libc/src/Math.cpp)

- [sincos](../../../core/libs/prx/libc/src/Math.cpp) (libc) follows FreeBSD 11.4.0 msun [`s_sincos.c`](https://github.com/freebsd/freebsd-src/blob/release/11.4.0/lib/msun/src/s_sincos.c), sharing reduction and `k_sin.c`/`k_cos.c` with `sin`/`cos` ([`k_sincos.h`](https://github.com/freebsd/freebsd-src/blob/release/11.4.0/lib/msun/src/k_sincos.h)). PPSA07923/PPSA05916 `libc.prx` gives `sincos` identical bits and `errno` as `sin`/`cos` over 300k inputs ([measured](https://github.com/boykopovar/AnyPS5/pull/16740#issuecomment-6718722071)); FP exceptions unmeasured. Unlike upstream, sets `EDOM` when high word is `0x7ff00000`/`0xfff00000` (other NaNs leave `errno`), matching `sin`/`cos`/`sincosf`. Verified vs upstream without FMA contraction; FMA-contracted builds may differ.
