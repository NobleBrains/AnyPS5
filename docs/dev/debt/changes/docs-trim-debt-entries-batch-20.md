# libSceNKWebKit

## change [Inspector::ScriptCallFrame](../../../core/libs/prx/libSceNKWebKit/src/InspectorScriptCallFrame.cpp)

- [Inspector::ScriptCallFrame](../../../core/libs/prx/libSceNKWebKit/src/InspectorScriptCallFrame.cpp), [ScriptCallStack](../../../core/libs/prx/libSceNKWebKit/src/InspectorScriptCallStack.cpp), [AsyncStackTrace](../../../core/libs/prx/libSceNKWebKit/src/InspectorAsyncStackTrace.cpp) (40/40/32B) follow [PS5 11.x WebKit](https://github.com/neonmodder123/PS5-WebKit-11.x/tree/80f77e94c63ee7202eec3d4b5b1a021dd89011ba/Source/JavaScriptCore/inspector), unverified on console. [FieldGetters.cpp](../../../core/libs/prx/libSceNKWebKit/src/FieldGetters.cpp) uses frame offsets `sourceID` +24 (`uint32_t` `JSC::SourceID`), `lineNumber` +28, `columnNumber` +32. OOB `ScriptCallStack::at`, wrapping refcounts, childless removals, bufferless non-empty frame vectors, and nulls throw instead of wrapping/OOB reads. `buildInspectorObject`/`buildInspectorArray` throw without `JSON`. Closes: console check + `JSON`.

# libSceNet

## change [sceNetSetsockopt](../../../core/libs/prx/libSceNet/Export.cpp) (libSceNet) stores

- [sceNetSetsockopt](../../../core/libs/prx/libSceNet/Export.cpp) (libSceNet) stores `SO_ONESBCAST` (`0x10000`, [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/net/net_types.h) `ORBIS_NET_SO_ONESBCAST`) and `sceNetGetsockopt` returns it (PPSA34501 crashes on `EOPNOTSUPP`). Sends ignore it (host routing, unrewritten dst). FreeBSD 11.4 [ip(4)](https://man.freebsd.org/cgi/man.cgi?manpath=FreeBSD+11.4-RELEASE&query=ip&sektion=4) `IPPROTO_IP` `IP_ONESBCAST` rewrites directed broadcasts (`192.168.2.255`) on `SO_BROADCAST` sockets to `255.255.255.255`; shadPS4 ignores sends and its [TODO](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/net/net.cpp#L2467-L2470) assumes `255.255.255.255`->broadcast. Console `SOL_SOCKET` `0x10000` vs `IP_ONESBCAST` and FreeBSD 11.4 `sogetopt()` (`so_options & optname`, `sys/kern/uipc_socket.c`) vs stored value are unverified.

## change [sceNetResolverAbort and sceNetResolverStartNtoaMultipleRecords]

- [sceNetResolverAbort/StartNtoaMultipleRecords](../../../core/libs/prx/libSceNet/Export.cpp) follow shadPS4 ([net.cpp](https://github.com/shadps4-emu/shadPS4/blob/811bfcd7d17ed8ae975c477d9ff28abc744e1ca8/src/core/libraries/net/net.cpp), [net_resolver.cpp](https://github.com/shadps4-emu/shadPS4/blob/811bfcd7d17ed8ae975c477d9ff28abc744e1ca8/src/core/libraries/net/net_resolver.cpp)); `Abort(int rid, int flags)` matches [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/Net/Socket.cs); unverified on console. Sync host lookups cannot be interrupted: active abort returns at once, ignores flags, and fails lookup `EINTR` (no output) on return. Idle flag 1 (NTOA) or 2 (ATON) fails next `StartNtoa*`/`StartAton` `EINTR` (`sceNetResolverGetError`) post-checks; other flags throw. `StartNtoaMultipleRecords` throws on any flag (1 async, `0x10000` no literals), like `Ex`.

# libSceNgs2.native

## change [sceNgs2SystemSetUserData, sceNgs2SystemGetUserData, sceNgs2SystemLock, sceNgs2SystemUnlock]

- [sceNgs2System{SetUserData,GetUserData,Lock,Unlock}](../../../core/libs/prx/libSceNgs2.native/src/System.cpp) (libSceNgs2.native) - shadPS4 checks `INVALID_SYSTEM_HANDLE` only, SharpProspero only names them; null user data throws. `Lock`/`Unlock` follow [prosper](https://github.com/mattias800/prosper/blob/d61eb2c0ad2de879ef74df9c4aa050c512804a18/prosper/src/hle/audio/hle_audio.cpp#L4367-L4384): Lock takes a recursive per-system lock; Unlock releases one hold from any thread (0 if unheld); `sceNgs2SystemRender` tries the lock and if held zeroes one grain per output, preserves render count/voices, and returns 0. Other calls do not wait on it (Peace Walker PPSA18216 spins holding it until `Render` and 8 `sceNgs2VoiceGetStateFlags`). Unknown if console voice/rack calls lock it; `sceNgs2RackLock` still holds the library lock until `Unlock`.

## change [sceNgs2GeomCalcListener]

- [sceNgs2GeomCalcListener](../../../core/libs/prx/libSceNgs2.native/src/Geom.cpp) builds an orthonormal world-to-listener row-major matrix (front priority, flag 1 right-handed) per [PS5PCEM](https://github.com/iStark/PS5PCEM/blob/d2534bac6325a2c61c0fa88977e5321074304484/src/hle/ngs2_geom.zig), unverified on hardware ([KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/df5b9b92be7dfd463b47756b707d6d356a9f0550/src/libs/ngs2.cpp) writes identity). Keeps world velocity and positive finite sound speed. Normalizes/orthogonalizes front/up; parallel vectors fall back to `(0,1,0)`/`(1,0,0)` for Dead Cells (PPSA15552, after sampler rack #4334) passing `orient_front={0,0,1}`, `orient_up={0,0,1}` (PS5PCEM rejects). Zero orientation, nonfinite input, unrepresentable translation, and nonpositive sound speed (PS5PCEM/KytyPS5 default 343) throw unchanged output; error codes unknown. No `sceNgs2GeomApply` attenuation/panning/Doppler.

# libSceNpAuth

## change [sceNpAuthAbortRequest, sceNpAuthGetAuthorizationCode, sceNpAuthGetAuthorizationCodeA, sceNpAuthGetAuthorizationCodeV3, sceNpAuthGetIdToken, sceNpAuthGetIdTokenA, sceNpAuthGetIdTokenV3, sceNpAuthPollAsync, sceNpAuthWaitAsync]

- [sceNpAuth{AbortRequest,Get{AuthorizationCode,IdToken}{,A,V3},{Poll,Wait}Async}](../../../core/libs/prx/libSceNpAuth/Export.cpp) - tracking, `0x80550306`/`0307`/`0301`/`0304` (`REQUEST_NOT_FOUND`/`INVALID_ID`/`INVALID_ARGUMENT`/`ABORTED`) follow [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/f68923fe807f657348e6719ce658bcbd0a2b1c72/src/core/libraries/np/np_auth/np_auth.cpp) (A->V3 `L189-193`/`L195-199`/`L375-378`/`L380-383`; `SCE_NP_ERROR_SIGNED_OUT` deferred async, sync immediate, no PSN). PPSA02225/PPSA15246(`0x4feefd`)/PPSA30601(`0x16edb90`): `0x20` code `(size,u32,clientId[37|128],const*)`->`(reqId,param,code[128],int* issuerId)`, `0x28` token `(size,u32,clientId,buf[256],const*)`->`(reqId,param,token)`, `CreateAsyncRequest`(`0x18`), `PollAsync`(`0` done,`<0` fail,`1` timeout->Abort after 41x100ms;`0x80550301`/`0x80550306`), `DeleteRequest`. Size checks/console unverified.

# libSceNpCppWebApi

## change [Activities::V1::UserActivities::Activity accessors]

- [Activities::V1::UserActivities::Activity accessors](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) return non-owning opaque refs for `getActivityId` (+0x10), `getName` (+0x30), `getMatchId` (+0x58), and booleans `getHidden` (+0x50), `matchIdIsSet` (+0x78), matching static inspection of PPSA21564 and PPSA05347 modules. Collection getters return by value via hidden RDI output (`self` in RSI); 24-byte vector handles keep the backing object at +0x30 (PPSA21564) vs +0x18 (PPSA05347), `UserActivities::getActivities` reads +0x30 in both, and `GetUsersActivitiesResponse::getUsers` reads +0x30 vs +0x10. Both share `0x0101` middle 16 descriptor bits, so those do not distinguish layouts. Model construction, strings, layout selection, other SDKs, and console behavior are unverified; collection getters still refuse.

# libSceNpEntitlementAccess

## change [sceNpEntitlementAccessGetGameTrialsFlag, sceNpEntitlementAccessGetPftFlag]

- [sceNpEntitlementAccessGetGameTrialsFlag, sceNpEntitlementAccessGetPftFlag](../../../core/libs/prx/libSceNpEntitlementAccess/Export.cpp) (libSceNpEntitlementAccess) - report full game (not trial/Play First Trial): write 0 to `uint32_t`/`SceNpEntitlementAccessGameTrialsFlag*` output and return 0, or `0x817D0002` on null. `GetGameTrialsFlag` follows [sharpemu](https://github.com/sharpemu/sharpemu/blob/a2983692930564815a77f81e74c151268f5b935f/src/SharpEmu.Libs/Np/NpEntitlementAccessExports.cs#L98-L116); `GetPftFlag` follows [dlc_emu](https://github.com/drakmor/dlc_emu/blob/main/src/dlc_modules/dlc_content.cpp). `0x817D0002` matches KytyPS5, prosper, and `sceNpEntitlementAccessGetEntitlementKey` (whereas `sceNpEntitlementAccessGetSkuFlag` returns `0x80558003`). Unverified on console; PRAGMATA (PPSA02530) has four live registers at the call, so extra arguments are not ruled out.

# libSceNpManager

## change [The account and sign-in profile getters]

- [The account and sign-in profile getters](../../../core/libs/prx/libSceNpManager/Export.cpp) (libSceNpManager) follow firmware 11.00 register layouts: `Sdk` variants and `GetAccountId`/`GetAccountType`/`GetAccountNpEnv`/`GetActiveSigninState`/`GetAuthServerErrorFlag`/`GetOnlineIdChangeFlag`/`GetPlusMemberTypeNB`/`GetServerError`/`GetSigninState`/`GetUserNum` take `(int userId, output)`; `GetNpEnv`/`GetVshClientId` take `(output)`; `GetOfflineAccountId` takes `(int userId, accountId, flag)` (both outputs optional). Rejecting `-1` or null required output with `0x80550003`, they return `0x80550006` (signed out; firmware returns `0x80550002` if manager absent). `GetUserIdByAccountId`/`GetOnlineIdByAccountId` (64-bit id, rejects 0), `GetUserIdByOnlineIdSdk`/`GetNpIdByOnlineId` (online id), and `GetNpIdSdk`/`GetOnlineIdSdk` (user id) also return `0x80550006`. `GetAccountType` zeroes output first; `GetNpEnv` zeroes 0x18 bytes.

## change [The Push and Push2 lifecycle]

- [The Push and Push2 lifecycle](../../../core/libs/prx/libSceNpManager/Push.cpp) (libSceNpManager) follows firmware 11.00 `libSceNpManager.sprx`: `sceNpPushInit` (`0x8055A001` if initialized), `sceNpPushTerm` (`0x8055A002` if uninit), `sceNpPushCheckCallback` (`0x8055A002` or 0, no queued events). `sceNpPush2Init` takes a 0x18-byte block (+0x10 <= 1, +0x14 in 1..5) and allocates 1-based context slots up to 32 (`0x8055C005` if full); `sceNpPush2Term` clears one context and returns 0. `sceNpPush2IsInit`, `sceNpPush2SetGlobalMutex`, `sceNpPush2OptionalCheckCallback`, and `sceNpPush2TriggerEmptyUserEvent` handle state queries; `sceNpPushIntGetConnectionState` writes 0 (disconnected). Null-arg `0x8055A003` is inferred; `0x8055C002`, `0x8055C003`, `0x8055C005` are measured. Notification transport, callbacks, data types, filters, comm id, user contexts, start/stop, and inactivity calls still throw.

## change [NpOnlineId, NpId, User, NpTitleToken, NpAccessToken and NpTicket value methods]

- [NpOnlineId, NpId, User, NpTitleToken, NpAccessToken and NpTicket value methods](../../../core/libs/prx/libSceNpManager/ValueClasses.cpp) - ctors, `Clear`, cmp, `SetNpOnlineIdStr`, `GetStr`, `IsEmpty`, `GetUserId`, `User` lookups follow fw 11.00 `libSceNpManager.sprx`: `NpOnlineId` (16B + u32 term @0x18), `NpId` (0x24B SDK), `User` (u32 id), `NpTitleToken` (0x410B), `NpAccessToken` (0x108B). `SetNpOnlineIdStr` rejects >16 chars (`0x80559E11`), copying measured length vs fw 16B read. Account lookups/`IsLoggedIn` call `sceUserServiceGetNpAccountId`; `GetUser` checks `sceUserServiceGetRegisteredUserIdList` (16 vs fw 20), returning `0x80559E2A` if invalid; `GetAccountId` returns `0x80559E2B` for 0; `GetNpOnlineId` returns `0x80550007` (invalid user) or `0x80550006` (signed out). `NpTicket::GetTicketData` throws on null (owner @0x08, data @0x10, size @0x18).

## change [NP state callbacks]

- [NP state callbacks](../../../core/libs/prx/libSceNpManager/Export.cpp) - `sceNpRegisterStateCallbackA` (8 slots, slot+1) and `sceNpRegisterStateCallback` (1 slot) follow [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/15abf68c9b52c4b04d094f2837719fdb871021f3/src/core/libraries/np/np_manager.cpp) (`L857-901`,`L981-1005`): `INVALID_ARGUMENT` (null), `CALLBACK_ALREADY_REGISTERED`, `CALLBACK_MAX` (`0x8055001D`, [np_error.h](https://github.com/shadps4-emu/shadPS4/blob/15abf68c9b52c4b04d094f2837719fdb871021f3/src/core/libraries/np/np_error.h#L38)). Sends `SIGNED_OUT` once for user `0x10000000` on next `sceNpCheckCallback` as `(user,SIGNED_OUT,nullptr,userdata)` (`L823`,`L903-962`; A matches [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/9e41d8320a2774b303fbad344c2347e8d0292e83/src/libs/network.cpp) `L3959-3974`,`L4009-4034`; reachability skips `UNAVAILABLE` vs `L922-961`). Unregister mid-dispatch runs once; console unmeasured.

# libSceNpTrophy2

## change [Extraction actions]

- [Extraction actions](../../../core/libs/prx/libSceNpTrophy2/src/TrophyPackage.cpp): conversions and extra action/output fields lack verified semantics. Rules with action members besides `input`/`output` or output members besides `statId`/`statName`/`sourceId` (including metadata or explicit null) are diagnosed and skipped instead of copying input. `statName`/`sourceId` are label-checked against the `statId` definition or skipped; across 116 titles' `udsNN.ucp` files, all 5648 matching outputs equal their stat definition (386 rules otherwise supported, 334 writing a trophy `udsStatId`). Console handling of mismatched labels is unknown. Conversion rules in [#8005](https://github.com/boykopovar/AnyPS5/issues/8005#issuecomment-6317178249) lack public shapes/measurements; no compatibility impact established.

## change [Trophy progress statistics]

- [Trophy progress statistics](../../../core/libs/prx/libSceNpTrophy2/src/TrophyPackage.cpp): `max` (from `initialValue` or `minValue`), `int32` (0..2147483647), and rule conditions (`eventName` + one `property` with `==` uint or `>` int, rejecting strings/non-integers) follow shipped title rules, unverified on console (`latest`/unsigned follow KytyPS5). Across 116 titles' `udsNN.ucp` files, only `==` and `>` appear: Demon's Souls (PPSA01341) uses `DsCollection` `> -1` for stats 152-154; Horizon Call of the Mountain (PPSA05666) uses `> 0`; Borderlands 4 (PPSA01494) feeds `udsStatId` from `$._trophy_progress` of `_UpdateTrophyProgress` via `int32` `max` and `$._trophy_id`. In PPSA12544, `readStat`/`readRule` skip/drop 22 rules (`[trophy]` logged) targeting `collection` (stat 1, 4 rules) or `count` (stats 114-131, 18 rules), leaving any `udsStatId` trophy referencing them at no progress.

## change [Damaged records]

- [Damaged records](../../../core/libs/prx/libSceNpTrophy2/src/TrophyStore.cpp): any line in `_trophy/<user id>_<service label>.txt` or `.stats.txt` that is not a new `<id> <value>` pair (blank, CR, duplicate/negative id, non-numeric value) makes `readRecord` throw `std::runtime_error` (naming file and line) on every read until fixed/deleted, halting the title (main returned 0). Unlock records are read by `sceNpTrophy2GetTrophyInfo`, `GetTrophyInfoArray`, `GetGameInfo` (with context), `GetGroupInfo`, `GetGroupInfoArray` (with loaded data), non-refused `_UnlockTrophy` via `sceNpUniversalDataSystemPostEvent`, and stat changes (with loaded data); stat records are read by rule-matched `PostEvent` and progressive trophy queries. Console behavior is unknown.

# libSceNpUniversalDataSystem

## change [Guest pointer fallbacks]

- [Guest pointer fallbacks](../../../core/libs/prx/libSceNpUniversalDataSystem/src/Event.cpp) (libSceNpUniversalDataSystem) - console behavior for unreadable params/names/buffers or unwritable outputs is unmeasured. As a host-safety fallback preserving `errno`, `sceNpUniversalDataSystem{,Int}Initialize` (bad param block), `CreateContext`, `CreateHandle`, `CreateEvent`, `CreateEventProperty{Array,Object}`, `EventProperty{Array,Object}Set*` value outputs, `EventEstimateSize`, `EventToString`, and `Int` record creators/setters return invalid-argument without registering state if an output write fails. Unreadable or >=4096-byte event names return invalid-event-name (host limit). `EventEstimateSize`/`EventToString` return null-event for unknown event pointers.

# libSceNpUtility

## change (../../../core/libs/prx/libSceNpUtility/Export.cpp) - signatures and errors from the wrappers

- [sceNpServiceChecker2Int{AbortRequest,CreateRequest,DestroyRequest,Finalize,GetServiceAvailability,GetServiceAvailabilityA,Initialize,IsSetServiceType}](../../../core/libs/prx/libSceNpUtility/Export.cpp) - signatures/errors from 11.00 `libSceNpUtility.sprx` (SHA-256 `1c90050abb737d3290539f46928fb536faca8773e335523479f06f4ae33f0c60`) wrappers at `0x14670`, `0x146e0`, `0x14710`, `0x14780`, `0x147a0`, `0x14840`, `0x148e0`, `0x148f0`: Initialize/Finalize manage host state; CreateRequest assigns ids >=1 (caps at `0x1000` live requests -> `0x80550706`, host limit, ids increase skipping live); Destroy ignores unknown ids (0), while Abort and availability calls return `0x8055B304` or `SIGNED_OUT` offline without writing; `IsSetServiceType` reads bits 0..9. Wrapper immediates give `0x8055B301`, `0x8055B302`, `0x8055B304`, `0x8055B305`; console payload, offline error, and request limit are unmeasured.

# libScePlayGo

## change [libScePlayGo](../../../core/libs/prx/libScePlayGo/Export.cpp) - the chunk set comes from

- [libScePlayGo](../../../core/libs/prx/libScePlayGo/Export.cpp) - chunk set reads `playgo-chunkdefs.xml` (listed + 0..default) or `sce_sys/playgo-chunk.dat` (`plgx` u16 count @0x0A, 0x20B entries @0x100 on PPSA01599:35, PPSA05110:25, PPSA14632/PPSA01802:1; assumes contiguous ids 0..count-1), reporting `LOCAL_FAST`. With `sce_sys/playgo-scenario.json` (PPSA01748) or no `playgo-*` (PPSA03001, PPSA03490, PPSA01285), ids 0..999 report `LOCAL_FAST` (`scePlayGoGetChunkId` lists them for scenario.json; PPSA15388 asks 0..999, PPSA31327 asks 0..5,10). Above 999: scenario.json throws unless `APS5_PLAYGO_UNLISTED_CHUNKS`=`installed` (`LOCAL_FAST`) or `absent` (`SCE_PLAYGO_ERROR_BAD_CHUNK_ID`, ending PPSA02630's u16 walk); no-`playgo-*` returns `BAD_CHUNK_ID` (ending PPSA03001's walk). Other `playgo-*` (`playgo-ficm.dat`/`playgo-hash-table.dat`) alone, or non-`plgx`/`plgo`/zero count, throw. Unverified on console.

# libScePlayReady4

## change scePlayReadyApiPlatform, scePlayReadyCdmiClose

- [scePlayReady{ApiPlatform,LicenseCount,Cdmi{Close,CreateMediaKey{Session,SystemAccess,s},Decrypt{Multiple,}Opaque{,2},DestroyMediaKey{Session,SystemAccess,s},FreeOpaqueDecryptedContent,GenerateRequest,Get{Configuration,Expiration,KeySystem,MediaKey{Count,Status},Session{AppContext,Id}},IsLicenseAckRequired,Load,Remove,RetrieveOpaqueDecryptedContent,Set{ServerCertificate,SessionCallbacks},Update},Profile{Clear,Enter,Leave,Pause,Resume,Start,Stop}}](../../../core/libs/prx/libScePlayReady4/Export.cpp) (libScePlayReady4) - names/NIDs appear in [PS5 3.20 export inventory](https://github.com/DNNDHH/PS5-3.20_Libs/blob/b2a77c09cd71c01f34c790cfbe98c751a13262ad/libScePlayReady4.c), without credible prototypes. Zero-param declarations resolve imports only, not ABI; do not call until verified.

# libScePosixForWebKit

## change [mkostemp](../../../core/libs/prx/libScePosixForWebKit/TemporaryFile.cpp)

- [mkostemp](../../../core/libs/prx/libScePosixForWebKit/TemporaryFile.cpp) (NID `QP8dYkB8zCU`) - console ABI/behavior unverified; `O_DIRECT` passes through to libkernel `open` per FreeBSD 11.4 [`_gettemp`](https://github.com/freebsd/freebsd-src/blob/8c639945a445/lib/libc/stdio/mktemp.c#L109-L174) (`F_GETFL` reports it, host still uses page cache). Untranslatable `flock` lock failures throw; otherwise follows FreeBSD 11.4 [mktemp.c](https://github.com/freebsd/freebsd-src/blob/releng/11.4/lib/libc/stdio/mktemp.c) and [open-time locking](https://github.com/freebsd/freebsd-src/blob/releng/11.4/sys/kern/vfs_vnops.c): unknown flags return `EINVAL` before reading template, `O_EXLOCK` overrides `O_SHLOCK`, lock failure closes fd leaving file, null template throws after flag check. Shared `mkstemp`/`mkostemp` host-safety checks return `EFAULT` (14) on bad input/trailing-`X` or `ENOMEM` (12) on alloc failure.
