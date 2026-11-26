# libSceProprietaryVoiceChatHelperPrx

## change [libSceProprietaryVoiceChatHelperPrx]

- [libSceProprietaryVoiceChatHelperPrx](../../../core/libs/prx/libSceProprietaryVoiceChatHelperPrx/Export.cpp) implements only the no-system-voice-chat path: `sceProprietaryVoiceChatHelperInitialize` and `sceProprietaryVoiceChatHelperTerminate` only track init state, and `sceProprietaryVoiceChatHelperSetVoiceChatState` returns success as a no-op (state arg unknown, nothing recorded). `sceProprietaryVoiceChatHelperGetVoiceChat` and `sceProprietaryVoiceChatHelperGetVoiceChatUsageState` throw because no usage state is recorded and return/out-pointer layout and encoding are unknown. Double init and calls outside the init/terminate window throw (console error codes unknown; whether console checks init order is unverified).

# libSceRtc

## change [sceRtcFormatRFC3339, sceRtcFormatRFC2822, sceRtcFormatRFC3339Precise]

- [sceRtcFormatRFC3339, sceRtcFormatRFC2822, sceRtcFormatRFC3339Precise](../../../core/libs/prx/libSceRtc/Export.cpp) (libSceRtc) - shifted ticks convert to 16-bit-year `SceRtcDateTime` wrapping past 65535 (10000/65535/65536 -> `0x80B50008`, 65537 -> 0001 + wrapped RFC 2822 weekday); `Precise` returns `0x80B50003` on 64-bit eastern overflow. FW 13.42 module: year rule for SDK `0x01000051`/`0x03000000`/`0x0A000000`, overflow `0x80B50003` for `0x03000000`/`0x0A000000` (`0x01000051` wraps). FW 11.60 PS5 and FW 13.60 PS5 Pro payloads (title diffs unmeasured) matched wrap and zones (+840 yr 1 -> `+14:00`; -720/±2000/extreme Precise -> `0x80B50003`); 13.60 `GetDayOfWeek` checks month (`0x80B50009`) before year (`0x80B50008` <1/>9999; `0x80B5000A` day), and negative ticks return `0x80B50003` (RFC2822) vs `0x80B50008` (RFC3339).

# libSceRudp

## change [sceRudpGetStatus]

- [sceRudpGetStatus](../../../core/libs/prx/libSceRudp/Export.cpp) - PPSA01521 passes out-ptr and size `0xF8`. PS5 (FW 13.60): status is `0xF8` bytes; before `sceRudpInit` returns `RUDP_ERROR_NOT_INITIALIZED`, after init null ptr or size 0/>`0xF8` returns `RUDP_ERROR_INVALID_ARGUMENT`. Layout matches `SceRudpStatus` in [Vita3K](https://github.com/AndnixSH/Vita3K-PS-Vita-OS-Fork/blob/master/vita3k/modules/SceRudp/SceLibRudp.cpp) (`currentContexts` at `0x64`; `allocs`, `frees`, `memCurrent`, `memPeak` at `0xC8`–`0xD4`). Host reports only `currentContexts` (traffic and allocator counters stay 0; console reports 6 allocs / `0x1260` B after `sceRudpInit`, plus 4 allocs / `0x3F0` B per context) and writes only the requested byte count (console writes those counters even if byte count is smaller than their offset).

# libSceSaveData.native

## change [sceSaveDataInitialize3]

- [sceSaveDataInitialize3](../../../core/libs/prx/libSceSaveData.native/Export.cpp) (libSceSaveData.native) - repeated inits succeed and are reference-counted until the final `sceSaveDataTerminate`. PS5 (FW 11.60 payload; title may differ) returns `0x809f0001` for terminate before init (as here), but `sceSaveDataInitialize3(NULL)` hangs so pairing is unknown. In PPSA12544 (whose Unity SaveData plugin fails on any error), the exe calls it once (`0x1721d8f`, arg `0`, guarded by a flag byte, no `sceSaveDataTerminate` import) and the plugin calls it once (`0x1fd2`, arg `0`, flag-guarded) and `sceSaveDataTerminate` at `0x2059`/`0x20d9`. Across 53 local titles (tail jumps counted, indirect ignored), all call it and 28 import `sceSaveDataTerminate` (17 both 1x, 10 both 2x, 1 init 1x / terminate 2x).

## change The following pending declarations have unverified signatures and behavior

- [53 pending `sceSaveData*` declarations](../../../core/libs/prx/libSceSaveData.native/Export.cpp) (`sceSaveDataBindPsnAccount`..`sceSaveDataUpload`: `BindPsnAccount*`, `Change*`, `Check*`, `CreateUploadData`, `Delete*`, `DirNameSearchInternal`, `Download`, `Get*`, `InitializeForCdlg`, `Is*`, `Manual*`, `Mount5`/`MountInternal`/`MountSys`, `RebuildDatabase`, `Restore*`, `Set*`, `ShutdownStart`, `SupportedFakeBrokenStatus`, `SyncCloudList`, `UmountSys`) are parameterless import-resolution stubs with unverified signatures/behavior that throw `NotImplemented_nid_no_patch`.

## change an unterminated or empty directory name

- [sceSaveDataMount3](../../../core/libs/prx/libSceSaveData.native/Export.cpp) - unterminated/empty dir, `.`, `..`, or `/`, `\`, `:` returns `SAVE_DATA_ERROR_PARAMETER` ([SDK 12.000](https://github.com/peterpuppy/bm/blob/3f9d1ba6b896a53080460c903f5c15af8affb80d/features/ps5/document/output/psn_12/System/Save_Data/SaveData-Reference/sce-save-data-mount3.md#L196-L204)), leaving result untouched like [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/569ec04d1a77127c601eed522ad5bba187173f9d/src/libs/libSaveData.cpp#L652-L658). [SDK](https://github.com/peterpuppy/bm/blob/3f9d1ba6b896a53080460c903f5c15af8affb80d/features/ps5/document/output/psn_12/System/Save_Data/SaveData-Reference/sce-save-data-dir-name.md#L32-L36) allows A-Za-z0-9`-.@`; KytyPS5 ([`valid_path_component`](https://github.com/KytyPS5/KytyPS5/blob/569ec04d1a77127c601eed522ad5bba187173f9d/src/libs/libSaveData.cpp#L276-L287)) adds `_`; other console rejections unchecked.

# libSceSsl

## change [sceSslCreateConnection, sceSslDeleteConnection, sceSslSetVerifyCallback, sceSslConnect, sceSslRead, sceSslWrite]

- [sceSslCreateConnection, sceSslDeleteConnection, sceSslSetVerifyCallback, sceSslConnect, sceSslRead, sceSslWrite](../../../core/libs/prx/libSceSsl/Export.cpp) - ffmpeg.prx, [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/60977b29103452b9681ae10c7a9011d44965bf16/src/SharpProspero/Interop/Net/Http.cs#L195-L252), Mbed TLS 3.6.7 (TLS 1.2, 12 suites), FW 13.40/13.59 payloads ([#11922](https://github.com/boykopovar/AnyPS5/pull/11922); title/23.8s omitted). 0x8095F006 id/2nd Term; 0x8095F007 host/flags(Rd0..3,Wr0); 0x80410109 !sock; 0x80020016 Init(0); 0x809517D5 16K OOM(throws; 5568/12000/80544B); 0x8095F00C CA; 0x8095F010 TLS1.3; 0x8095F0FD cb<0(0x20/0x4/date; handles till Term); 0x8095F013/0x8095F014 block/RCVTIMEO(32/64K wr); 0x80410136/0x80410120 reset; 0x80951D55 MAC; 0x80951D78 !TLS; 0x8095170D hs EOF/alert wr. No 0x80951710/0x8095177C/0x8095F001; EOF=0; IP !SNI; Delete keeps sock; R/W locked; 0x8095F00A/unknown throw.

## change [sceSslClose]

- [sceSslClose](../../../core/libs/prx/libSceSsl/Export.cpp) (libSceSsl) - takes conn id ([SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/60977b29103452b9681ae10c7a9011d44965bf16/src/SharpProspero/Interop/Net/Http.cs#L195-L252)); measured on FW 13.40. Sends `close_notify` once post-handshake and returns 0 (also on repeat, unconnected, failed-verify, or fatal-alerted conn; pre-handshake close still allows connect). Post-close `sceSslConnect`/`sceSslRead` return 0; `sceSslWrite` throws (console code unmeasured). Non-blocking would-block returns `SSL_ERROR_WANT_POLLOUT` (`0x8095F014`, repeatable); post-reset returns `SCE_NET_ERROR_ECONNRESET` (`0x80410136`); unknown conn returns `SSL_ERROR_INVALID_ID` (`0x8095F006`). Leaves socket open (console behavior unknown).

## change [sceSslLoadCert, sceSslGetCaList, sceSslFreeCaList, sceSslGetCaCerts, sceSslFreeCaCerts, sceSslUnloadCert]

- [sceSslLoadCert, sceSslGetCaList, sceSslFreeCaList, sceSslGetCaCerts, sceSslFreeCaCerts, sceSslUnloadCert](../../../core/libs/prx/libSceSsl/Export.cpp) - [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/Net/Http.cs) layouts; PPSA15508/01.002.000 calls `sceSslLoadCert` (1,408-B PEM CA, null client cert/key). Mbed TLS parses DER/PEM; CA loads commit atomically, client cert replaces prior identity, raw keys parse at TLS setup (partial chains, native key errors, incomplete identities unsupported). CA queries return owned snapshots valid post-unload; repeated append, snapshot lifetime, overhead, and error precedence unmeasured on PS5. Invalid/malformed/OOM/unsupported keys throw. No root bundle copied (`NOT_FOUND` on empty ctx). Null-out/invalid-ctx precedence, output preservation, and SDK 7.50 unload boundary recovered; foreign CA clear unchanged.

## change [sceSslEnableVerifyOption, sceSslDisableVerifyOption, sceSslSetMinSslVersion, sceSslSetAlpn, sceSslGetAlpnSelected]

- [sceSslEnableVerifyOption, sceSslDisableVerifyOption, sceSslSetMinSslVersion, sceSslSetAlpn, sceSslGetAlpnSelected](../../../core/libs/prx/libSceSsl/Export.cpp) - [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/60977b29103452b9681ae10c7a9011d44965bf16/src/SharpProspero/Interop/Net/Http.cs) signatures; PS5 FW 13.40 elfldr payload. Verify bits: SERVER_VERIFY 0x1, CLIENT_VERIFY 0x2 (init unknown, off), CN_CHECK 0x4, NOT_AFTER 0x8 (0x8095F00D), NOT_BEFORE 0x10 (0x8095F00E), KNOWN_CA 0x20; default 0x25; >0x3F -> 0x8095F007; clearing 0x1/0x4/0x20 hit 0xa0020101 in payload (title unknown, host allows). SetMinSslVersion takes 2..5 (0/1/>5 -> 0x8095F007; TLS 1.2 only). SetAlpn splits on ',' (1-255 B each, <65535 B; empty -> 0x80951773; null/fail clears); GetAlpnSelected returns conn-owned ptr (valid post-close) or null. Bad conn/ctx -> 0x8095F006 (null for Get).

# libSceSysmodule

## change [sceSysmoduleLoadModuleByNameInternal]

- [sceSysmoduleLoadModuleByNameInternal](../../../core/libs/prx/libSceSysmodule/Export.cpp) (libSceSysmodule) - verified vs FW 4.03 `libSceSysmodule.sprx`: null `fname` returns `SCE_KERNEL_ERROR_EINVAL`; otherwise tries `/<root>/<tier>/lib/<fname>.<ext>` via `sceKernelLoadStartModuleForSysmodule` (roots: `sceKernelGetFsSandboxRandomWord`, `system_ex`, `system`; tiers: `priv_ex`, `common_ex`, `priv`, `common`; `.sprx` before `.prx`), returning first positive result or last error and forwarding `args`, `argp`, `flags`, `res` (null options). Firmware ORs `0x10000` to bypass `sceKernelLoadStartModule`'s flags==0 check, whereas host rejects nonzero `flags`. Host maps only `/<sandbox>/common/lib/` and `/system/common/lib/` to `libs/<name>.prx`, skipping missing paths as on console.

# libSceSystemGesture

## change [sceSystemGestureSetTouchAreaInformation]

- [sceSystemGestureSetTouchAreaInformation](../../../core/libs/prx/libSceSystemGesture/Export.cpp) (libSceSystemGesture) - NID `dWKhDgKLsHY` (`dWKhDgKLsHY#D#A` in 3.20/13.60 modules; 24 NIDs, sorted-LF SHA-256 `acad8b5f908d6431db3c2d2a5dcffbd79eca1e1f8a02127e8671d82da0fca38d`). From FW 11.00 `libSceSystemGesture.sprx` (SHA-256 `98425066e5c44346fc849931f62e369cad70e358e89ccadfa75eca2c5b1425f8`, not console-measured): export at `0x6ee0` (340 B) takes `sceSystemGestureOpen` handle and 32-byte struct, returning `0x80890003` uninit (`0xeaf0`), `0x80890001` null struct, `0x80890006` handle outside 8 slots (`0xeb00`–`0xeb1c`), copying 32 B (`0x6fcc`) and calling `0x5550` (`0x6fd9`) to set `0xc070 + index * 0x550 + 4`. Host stores the 32 B without applying them to recognition; flag effect is unmeasured.

# libSceSystemLogger

## change [sceSystemLoggerInitialize, sceSystemLoggerSetLogData and sceSystemLoggerTerminate]

- [sceSystemLoggerInitialize, sceSystemLoggerSetLogData and sceSystemLoggerTerminate](../../../core/libs/prx/libSceSystemLogger/Export.cpp) (libSceSystemLogger) - checks follow FW 11.00 `libSceSystemLogger.sprx` (SHA-256 `8b0788ae7885f085fa2a7eeafedf72f9c0352e8363a79afff06e56fe30e17292`; `0x700`, `0x730`, `0x760`), not console-measured: `Initialize` returns `0x81080005` when active; `Terminate` returns `0x81080004` when inactive; `SetLogData` takes `(tag, count, entries)` (4th ignored), rejecting tag >15 B or non-`[0-9A-Za-z]`, count >32, nonzero count with null ptr, or 100-B entry (64-B payload `0x00`, u32 len `0x40`, 32-B tail `0x44`) with len >64 or nonzero tail. Empty tag (incl. 0-len) and 0 count accepted; tag copied <=15 B to stderr. Service, ring buffer, transport, helper objects, and `0x81080001` paths unimplemented (always succeed).

# libSceSystemService

## change [sceSystemServiceGetAppId, sceSystemServiceGetAppTitleId, sceLncUtilGetAppTitleId]

- [sceSystemServiceGetAppId, sceSystemServiceGetAppTitleId, sceLncUtilGetAppTitleId](../../../core/libs/prx/libSceSystemService/Export.cpp) (libSceSystemService) - non-running app/title IDs, null buffers, and >=10-char running title IDs (10-B buffer) throw (console errors unknown). Null or len!=9 title ID makes `sceSystemServiceGetAppId` return -1 before lookup, matching FW 13.42 `libSceSystemService.sprx` ([emulator run](https://github.com/boykopovar/AnyPS5/pull/10801#issuecomment-6403075212)); otherwise compares against `param.json`/`param.sfo`. Signatures follow [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/SystemService/SystemService.cs#L236-L260), except `sceLncUtilGetAppTitleId` returns `int` (0 on success); `ErrorNoAppInfo` (`0x80A10009`) is named only for `sceSystemServiceGetAppFocusedAppStatus`.

## change console language numbers 0 to 29

- [sceSystemServiceParamGetInt](../../../core/libs/prx/libSceSystemService/Export.cpp) - languages 0–29 follow [OpenOrbis](https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain/blob/06fcf7fd108fa080035cdd3259ed22eb07e0508c/include/orbis/_types/sys_service.h#L54-L83) (21 Arabic), 30 (Ukrainian) follows PPSA99505 ([Jelly5](https://github.com/02dnot/Jelly5/blob/552b866d89b2a58b27f7f7b0484dc743ce235526/app/src/app/i18n.cpp#L29-L60)) and PPSA99002 ([ProsperoLight](https://github.com/blackbearreloaded/ProsperoLight/blob/b708bbde3457bef1073a9ab6959957cb691e8e2f/include/i18n.hpp#L252-L309)); unconfirmed on console (neither runs on `main`). Param 400 (name unknown; absent in 4.03) mirrors 1 (PPSA01325 tries 400 then 1 over 31 langs); 10.01/12.60 omit SDK-version mapping and return 0x80a10003 on an unknown check (neither modelled). Without ANYPS5_LANGUAGE/ANYPS5_DATE_FORMAT/ANYPS5_TIME_FORMAT: en-US, 1, 1.

## change the time zone (parameter 4) and summer time (parameter 5)

- [sceSystemServiceParamGetInt](../../../core/libs/prx/libSceSystemService/Export.cpp) (libSceSystemService) - time zone (param 4) and summer time (param 5) track host local time at call; titles add UTC + tz_min + (60 if summer==1) ([Apollo](https://github.com/bucanero/apollo-ps4/blob/252a9d5a187c463994263009851a179f4f910c49/source/exec_cmd.c#L70-L92), [GoldHEN's frame logger](https://github.com/GoldHEN/GoldHEN_Plugins_Repository/blob/6c3a06f53d9caebadfb0aeed9bb94119012010a7/plugin_src/frame_logger/source/main.c#L60-L82), [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/413d2e3518fb435f40971d238fe0ecae7ee551da/src/SharpProspero/Platform/SystemParameters.cs#L75-L79)), so tz subtracts active DST (60 min on Linux; CRT `_get_dstbias` on Windows). For non-1h DST, Linux preserves sum over standard offset and Windows the reverse; console behavior unknown.

## change [sceSystemServiceLoadExec]

- [sceSystemServiceLoadExec](../../../core/libs/prx/libSceSystemService/Export.cpp) - "exit"/converted /app0 exes (APS5_EBOOT, picker.picker.bin); skips atexit/dtors (stderr may drop). Argv omits exe ([LoadExec.cpp](../../../core/libs/prx/libSceSystemService/LoadExec.cpp)): PPSA85300 ([VLC](https://github.com/ZiZc3/VLC/blob/86bb7d474a303f62bb3c2fc12f728db4846085b8/src/platform_ps5.c#L565-L571)) empty; PPSA99169 ([1](https://github.com/mihawk-99/PS5_RetroArch/blob/bead80725cc11e3c5c65bd9424b0a4494ae36d45/frontends/picker/picker.cpp#L444-L446)/[2](https://github.com/mihawk-99/PS5_RetroArch/blob/bead80725cc11e3c5c65bd9424b0a4494ae36d45/src/relaunch_ps5.cpp#L161-L164)/[3](https://github.com/mihawk-99/PS5_RetroArch/blob/bead80725cc11e3c5c65bd9424b0a4494ae36d45/evidence/loadexec-second-image/relaunch-test.jsonl)) --ps5-relaunch=1/--ps5-mode=retroarch/"". Uses fexecve/POSIX_SPAWN_SETEXEC/wait (execv/CreateProcessW if null); limits/errors/!app0 throw.

## change [Running application ID]

- [Running application ID](../../../core/libs/prx/libSceSystemService/Export.cpp) (libSceSystemService) - `sceSystemServiceGetAppIdOfRunningBigApp` returns `1` (AnyPS5 placeholder; console shell IDs unknown). `sceSystemServiceKillApp(1, -1, 0, 0)` (as in [SwanStationPS5](https://github.com/darkxex/SwanStationPS5/blob/4cdc981d1a64ffb7f2af73f7a27a9e91a3cb5eb4/src/main.c#L1050-L1058) and [ps5-payload-dev's hbldr](https://github.com/ps5-payload-dev/websrv/blob/46374edee6627885abac717de24da7a1d73af6f1/src/ps5/hbldr.c#L425-L435)) runs guest shutdown and `atexit`, then exits 0 without returning (console returns and shell terminates; handler execution unknown). Other IDs/args throw.

# libSceUpdateService

## change sceUpsrvGetCompletedTaskId, sceUpsrvGetRebootFlag, sceUpsrvGetSystemExUpdateTaskState

- [40 `sceUpsrv*` exports](../../../core/libs/prx/libSceUpdateService/Export.cpp) (libSceUpdateService; `sceUpsrvGetCompletedTaskId`, `sceUpsrvGetRebootFlag`, `sceUpsrvInitialize`, `sceUpsrvTerminate`, `sceUpsrvStartUpdateSession`, `sceUpsrvUpdate*`, `sceUpsrv*SystemEx*` through `sceUpsrvVerifySystemExBank`) - names/NIDs are in [PS5 3.20 export inventory](https://github.com/DNNDHH/PS5-3.20_Libs/blob/b2a77c09cd71c01f34c790cfbe98c751a13262ad/libSceUpdateService.c), with no credible prototypes; parameterless declarations only resolve imports (not an ABI claim; do not call until verified).

# libSceUsbStorage

## change [sceUsbStorageRegisterCallbackForMapAvailable, sceUsbStorageUnregisterCallbackForMapAvailable, sceUsbStorageRegisterCallbackForMapUnavailable, sceUsbStorageUnregisterCallbackForMapUnavailable]

- [sceUsbStorageRegisterCallbackForMapAvailable, sceUsbStorageUnregisterCallbackForMapAvailable, sceUsbStorageRegisterCallbackForMapUnavailable, sceUsbStorageUnregisterCallbackForMapUnavailable](../../../core/libs/prx/libSceUsbStorage/Export.cpp) (libSceUsbStorage) - stores `(device, directory, callback)`; pre-`sceUsbStorageInit` or duplicate registration throws, unknown unregistration keeps marker. Dispatch matches stored directory (relative vs mount receives own user data). Map calls available callbacks `(1, device, flags)` (allows nested map); unmap or rescan drop calls unavailable `(2)`; `sceUsbStorageTerm` clears without callbacks. Keying, dir spelling, reentrancy, scan timing unmeasured; shadPS4 stubs all four.
