# libSceAgcDriver

## change [sceAgcDriverSetSubmitValidationMode, sceAgcDriverGetSubmitValidationMode, sceAgcDriverSetSubmitValidationConfig, sceAgcDriverGetSubmitValidationConfig, sceAgcDriverSetValidationErrorOutputFrequency, sceAgcDriverGetShaderDebuggingStatus]

- [sceAgcDriverSetSubmitValidationMode, sceAgcDriverGetSubmitValidationMode, sceAgcDriverSetSubmitValidationConfig, sceAgcDriverGetSubmitValidationConfig, sceAgcDriverSetValidationErrorOutputFrequency, sceAgcDriverGetShaderDebuggingStatus](../../../core/libs/prx/libSceAgcDriver/Unimplemented.cpp) (libSceAgcDriver) - the five validation calls return `0x8A6C1000` without reading args or writing caller memory, and `sceAgcDriverGetShaderDebuggingStatus` returns 1, as [OpenAGC](https://github.com/OpenAGC/OpenAGC/blob/3c7e8072f0d7684e930454794fd5c02056946d73/src/game_compat.c#L250-L291) reads Sony's libSceAgcDriver.sprx: each validation export is `mov eax, 0x8a6c1000; ret` in Dragon Quest VII Reimagined's bundled copy (0x7bb0-0x7bf0), debugging status 1 in FW 5.50 (0x6740). Parameters (32-bit mode or frequency, mode output, configuration pointer) follow OpenAGC and are never read. A PS5 Pro payload (FW 13.60) measured the same results; no known title calls them

## change [PM4 submission rings]

- [PM4 submission rings](../../../core/libs/prx/libSceAgcDriver/Execution/src/Driver/Queues/SubmissionRing.cpp) (libSceAgcDriver) keep only the first 512 dwords of each submission; later packets are not printed. The [failure report](../../../core/libs/prx/libc/src/FailureReport.cpp) dump avoids the C runtime heap: static packet names, integer/string conversions into a 512-byte stack buffer, one `WriteFile` per line. It still runs on whatever state the crash left. It waits ~100 yields for each ring's lock, else prints that queue as busy, so a crash while recording a submission loses that queue's ring. It reads the rings unchecked, so corrupted rings can fault inside the dump; such a fault while reporting an unhandled exception stops in that report's `Sleep(INFINITE)` and the process hangs. Reports run at most once per process. Linux has no crash report: rings are printed only by `APS5_STALL_WATCHDOG_SECS`.

## change [Comp-to-single DCC keys]

- [Comp-to-single DCC keys](../../../core/libs/prx/libSceAgcDriver/Graphics/src/DccMetadata.cpp) (libSceAgcDriver) - uniform key 0x10 (`GFX9_DCC_CLEAR_SINGLE` in Mesa `ac_descriptors.h`) sets every texel to the first element of its 256-byte block (where radv's `build_clear_dcc_comp_to_single_shader` writes the clear color). Sampled textures, storage images (marking keys uncompressed on first write), and CB DCC decompress passes expand it; fast-clear eliminate and mixed-key metadata passes throw, color targets use stored texels, and mixed keys or non-dividing formats read as stored. Uniformity checks one key byte per 256 surface bytes (see DCC key stores above), so pipe-aligned surfaces differing past that range still expand. Not verified on hardware

# libSceAppContent

## change [Additional content download and delete]

- [Additional content download and delete](../../../core/libs/prx/libSceAppContent/Export.cpp) (libSceAppContent) - downloading and deletion are unmodeled: `sceAppContentAddcontEnqueueDownload` answers `PARAMETER` for a null label (and a padded one from SDK 0x01500000), `sceAppContentAddcontDelete` for a null label (and a padded one from SDK 0x03500000), `sceAppContentGetAddcontDownloadProgress` for a null label or progress (and a padded label from SDK 0x03500000); each answers `NOT_FOUND` otherwise, as [dlc_emu](https://github.com/drakmor/dlc_emu/blob/fcf69b16d46a9850578bce19dc2c1b4dc867cbb5/src/dlc_modules/dlc_content.cpp#L2328-L2341) does for an uninstalled label. Layouts/prototypes follow [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/AppContent/AppContent.cs#L45-L54) and [dlc_emu](https://github.com/drakmor/dlc_emu/blob/fcf69b16d46a9850578bce19dc2c1b4dc867cbb5/src/dlc_modules/dlc_content.h#L91-L92).

# libSceAudioOut

## change [Legacy vibration ports]

- [Legacy vibration ports](../../../core/libs/prx/libSceAudioOut/src/AudioOut.cpp): PPSA99902's [input wrapper](https://github.com/Nazky/UnleashedRecomp-Prospero/blob/2ce067a1d51bb58670c7ee4a3aed893c504da805/UnleashedRecomp/hid/driver/sdl_hid.cpp#L316) opens type 10 with 256 stereo S16/F32 frames at 48000 Hz. Mono duplicates to both channels; wider layouts throw. Native output uses AudioOut2's 4-channel layout (6-channel on PulseAudio); without a controller sound card, the RMS motor transform runs on <=256-frame segments scheduled by queue frame count/rate, combining concurrent sources by per-motor max (drain/close remove only that port's segments) for non-DualSense pads. Unmeasured on hardware, not a waveform reproduction, single-controller routing only; AudioOut2 keeps its 80 ms peak hold.

# libSceAudiodec

## change [M4AAC extended parameters]

- [M4AAC extended parameters](../../../core/libs/prx/libSceAudiodec/Export.cpp): 28- and 32-byte variants add fields at offsets 24 (`0..1`, error `0x807F0305` `SCE_AUDIODEC_ERROR_M4AAC_INVALID_ENABLE_NONDELAY_OUTPUT`) and 28 (`0..2`, error `0x807F0306` `SCE_AUDIODEC_ERROR_M4AAC_INVALID_SURROUND_CHANNEL_INTERLEAVE_ORDER`, per fpPS4 `ps4_libsceaudiodec.pas`). Both non-delay 0 and 1 decode every AU immediately (shadPS4 holds 2 frames without `EnableNondelayOutput`; console delay for 0 is unmeasured). Interleave orders 1 and 2 leave mono/stereo unchanged (PPSA21607 uses order 2 for stereo) and throw on surround: mapping to shadPS4's `ExtlExtrLsRs`/`LsRsExtlExtr` and order 0 is unknown.

# libSceFont

## change [sceFontCharacterGetSyllableStringState]

- [sceFontCharacterGetSyllableStringState](../../../core/libs/prx/libSceFont/src/Text.cpp) (libSceFont) - signature `(textCharacter, int* syllableStringState)` per [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/main/src/SharpProspero/Interop/Font/SceFont.Engine.cs) (`SceFontSyllableString`: 0 NonDetection, 1 Starting, 2 Following, 3 Depending, 4 Connecting). PS4 FW 11.00 `libSceFont.sprx` (not PS5): state 0 for Latin, CJK, Hangul, Arabic, Hebrew, Devanagari and the other scripts tried; for Thai the library sets 1, 3 or 4 (never 2 on real text) while it creates the string; null character or result returns `SCE_FONT_ERROR_INVALID_PARAMETER`, and a null character writes 0 to a non-null result. Returns 0 outside Thai (U+0E00..U+0E7F) as [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/d2413fc2ebd91d1b7234197c2dd5bd48e1cb6a51/src/libs/libFont.cpp#L1805-L1815) does and throws for Thai, whose cluster states are not modelled. Called by PPSA06699 (Hot Shots Golf 2).

# libSceHmd2

## change [Hmd2BeforeInitializeError0004]

- [Hmd2BeforeInitializeError0004](../../../core/libs/prx/libSceHmd2/Export.cpp) (libSceHmd2) - before `sceHmd2Initialize` these 15 exports (names below drop `sceHmd2`) return `0x8a720004` with all-null or all-zeroed-16 KiB arguments, writing nothing: `kAMsjYHBbNQ` (InternalSetVolume), `5m73zmLpsQ0` (InternalResetViewPosition), `EPeH4xuQ2XQ` (InternalGetStatus), `TJ8lyRsjFY4` (InternalClearTrackerFts), `-XQVcLnyX68` (InternalSetSlamCommand), `giQwQoHhYYA` (InternalDfuStart), `zNc9IkgawMI`, `-tnG5UJPvAc` (GazeStartUserCalibration), `FPhFmG+jXuQ` (GazeStopUserCalibration), `CKlkvNngofQ`, `yL1CZaciFWE` (GazeSetUserCalibrationPoint), `qlzgyG3s0RY`, `kTIDrDLzMv0`, `9Gee0ef8a+U`, `ecZVjQdwmBI` (GazeSetEnabledEye). Measured in a FW 13.60 payload; a game process is assumed to match; arguments and post-init behaviour are unmeasured, and it throws once `sceHmd2Initialize` succeeded

## change [Hmd2BeforeInitializeMinusOne]

- [Hmd2BeforeInitializeMinusOne](../../../core/libs/prx/libSceHmd2/Export.cpp) (libSceHmd2) - before `sceHmd2Initialize` these 12 exports (names below drop `sceHmd2`) return -1 with all-null or all-zeroed-16 KiB arguments, writing nothing: `EAkv8Mqt33E` (InternalRelocalizerStart), `Rv7bkmKvmz0` (InternalRelocalizerStop), `1oPZizn0SNA` (InternalTrackingStart), `TwCpE-nNrlo` (InternalTrackingStop), `jkqAeQfdtxw` (InternalLedDetectorStart), `+l0RQph81Z8` (InternalLedDetectorStop), `qsmAtGw1Ncc` (InternalImageStart), `uKhwYoMsIsA` (InternalImageStop), `WNdaigr94dI` (InternalGenDataStart), `P6S5ZVWf-H0` (InternalGenDataStop), `sun6NWE1sdg` (InternalSlamTraceStart), `--TFsJYXo04` (InternalSlamTraceStop). Measured in a FW 13.60 payload; a game process is assumed to match; arguments and post-init behaviour are unmeasured, and it throws once `sceHmd2Initialize` succeeded

# libSceHttp

## change [sceHttpAbortWaitRequest, sceHttpWaitRequest]

- [sceHttpAbortWaitRequest, sceHttpWaitRequest](../../../core/libs/prx/libSceHttp/Export.cpp) (libSceHttp) - abort takes only the epoll handle, per [Orbital](https://github.com/AlexAltea/orbital/blob/master/tools/ida/db_types.json) and [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/main/src/SharpProspero/Interop/Net/Http.cs); behaviour/errors per [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/network/http.cpp) (PS4). On PS5 13.590.001 later waits and polls returned `SCE_HTTP_ERROR_ABORTED` (0x80431080); persistence and waking blocked waits unconfirmed. Waits woken by `sceHttpDestroyEpoll` or the last context's `sceHttpTerm` return 0 events. Epoll calls return `BEFORE_INIT` (0x80431001) without a live `sceHttpInit` context and `INVALID_ID` (0x80431100) for unknown contexts/handles; `sceHttpTerm` returns the same errors instead of succeeding for any id. Handles are pointers, so a stale handle reused by a newer epoll's address is taken as it

# libSceHttpCache

## change [sceHttpCacheSystemInit and sceHttpCacheSystemClearAll]

- [sceHttpCacheSystemInit and sceHttpCacheSystemClearAll](../../../core/libs/prx/libSceHttpCache/Export.cpp) - the wrappers at `0x2d570`/`0x2d680` in FW 11.00 `libSceHttpCache.sprx` (SHA-256 `0d825d639b642a73b3ae7d7e0217f602104609d388cf8e0ae83ba50a8a2c2fd1`) validate a non-null path of at most 0x400 bytes (`0x817a0012` on failure), pass it to their delegate and return 0 without checking it. `SystemInit` validates and returns 0 without storing; `SystemClearAll` validates, then throws via `NotImplemented_nid_no_patch`, since the delegate's effect on per-application caches is not established. The host copies the path once via `CopyGuestString_nid_no_patch`, so a bad pointer or unterminated string returns `0x817a0012`. The wrappers' path conversion is not reproduced (non-ASCII or invalid UTF-8 paths may be accepted), nor the on-disk layout. `SystemShutdown`, `SystemTerm`, `SystemSendStatistics` return 0 like their wrappers.

# libSceImeDialog

## change [sceImeDialogInit, sceImeDialogGetStatus, sceImeDialogGetResult, sceImeDialogAbort, sceImeDialogTerm]

- [sceImeDialogInit, sceImeDialogGetStatus, sceImeDialogGetResult, sceImeDialogAbort, sceImeDialogTerm](../../../core/libs/prx/libSceImeDialog/Export.cpp) (libSceImeDialog) - statuses (0-2), end statuses (0-2), and Init errors (`0x80bc0001`, `0x80bc0031`, `0x80bc0016`, `0x80bc0017`) follow [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/b2d7ec7d31f9ad9426db93afe079db8679b3e1e0/src/core/libraries/ime/ime_dialog.cpp) (only idle status 0 verified on PS5 11.60). GetResult returns `0x80bc0002` for null (shadPS4: `0x80bc0031`) or unmapped pointers (unmeasured host-safety fallback). Without an open dialog, Abort, Term, and valid-buffer GetResult return `0x80bc0107` writing nothing ([obSCEne](https://github.com/project-oops/obSCEne/blob/de5efc406319ade18df00654025494804405a7fb/reports/hardware/report-20261018-gt14-census.log#L530-L561), PS5 12.40). Abort on finished keeps `0x80bc0002` (unmeasured); Term closes running dialogs; accepted text writes once

# libSceNet

## change [sceNetGetNameToIndex and sceNetGetIfnameNumList]

- [sceNetGetNameToIndex and sceNetGetIfnameNumList](../../../core/libs/prx/libSceNet/Export.cpp) (libSceNet) - convert ids via `sceNetGetIfName`'s ten-name table: `sceNetGetNameToIndex` rejects `-1` with `EINVAL` and an id without host interface with `0x804101A4` (`ENODATA`); `sceNetGetIfnameNumList` rejects a null list, capacity <= 0 and flags > 1 with `EINVAL` and lists ids 1..8 (flags 0) or 1, 0, 2..9 (flags 1) up to capacity. A PS5 payload (11.60) got 2 from `sceNetGetNameToIndex(0)` before and after `sceNetInit`, and ids 1, 4, 5 (flags 0) and 1, 0, 4, 5 (flags 1); so only `sceNetGetIfnameNumList` checks init first (`0x804101c8`), as local `libSceNet.sprx` (build ID ad19de2df01d46df) does for both. Order of 1 and 0 without id 1 and other firmware unmeasured. The host has no FreeBSD interface names: id 0 is host loopback, 1 the first up non-loopback interface with an address, 2..9 a same-named host interface; the returned value is the host index.

# libSceNgs2.native

## change [NGS2 ATRAC9 streaming]

- [NGS2 ATRAC9 streaming](../../../core/libs/prx/libSceNgs2.native/src/Atrac9.cpp) keeps duration/skip counts across compressed blocks. `APPEND` (2, `ONLY_DATA` in ref) ignores continuation sample/skip/repeat counts per [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/bb98f26433fa9e711c219aac1711fe8ecba5bf3a/src/libs/ngs2.cpp). Partial superframes buffer across blocks; completion callbacks allow refills, with the last callback waiting for PCM drain. Buffering, callback timing, decoded-byte accounting, and starvation silence are unverified on hardware. Early-closed streams throw; fragmented repeats are unsupported; PCM retains existing behaviour.

# libSceNpCppWebApi

## change Vector<unsigned long>

- [sce::Np::CppWebApi::UserProfile::V1, ProfanityFilter::V2, CommunicationRestrictionStatus::V3](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) (libSceNpCppWebApi) - imported by an Epic Online Services module whose dump ships no `libSceNpCppWebApi.prx`. The 0x10-byte `ResponseHeaderBase` (count at 0x00, header block at 0x08) comes from PPSA21564/PPSA23732 `J03093737`/`J03341606` (`0x14980`, `0x149a0`, `0x149b0`); `getHeaderValue` returns 0x80553512 for a null block and throws when it is present. `WebApiFilterRequest` offset 0x00 is assumed. Offline, the five API calls, `WebApiFilterRequestFactory::create` and transaction `start`/`getResponseHeaders` fail with `SCE_NP_WEBAPI_ERROR_NOT_SIGNED_IN`; response accessors throw (no response exists). `Vector`/`Iterator` over `IntrusivePtr<UserProfile::V1::BasicPresence>` follow the bundled module on host-created bodies and throw on title ones. No host `Vector<unsigned long>` body exists, so `begin`, `end` and iterator throw when non-empty
