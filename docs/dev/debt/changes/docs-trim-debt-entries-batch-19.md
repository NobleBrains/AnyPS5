# libSceAudiodec

## change [libSceAudiodec]

- [libSceAudiodec](../../../core/libs/prx/libSceAudiodec/Export.cpp) accepts only 16-bit PCM (1), plus float PCM (2) for ATRAC9 and M4AAC, matching PS5 13.60 payload `sceAudiodecCreateDecoder` (`ERROR_INVALID_WORD_LENGTH` for `iBwPcm` 0, 3, -1 and MP3 float; other sizes assumed same). On PS4 10.71 (`iBwPcm` -1..3), MP3 accepts 1, M4AAC accepts 1 and 2, others return `ERROR_INVALID_WORD_LENGTH`; ATRAC9 with invalid config returns `ERROR_AT9_INVALID_CONFIG_DATA` for all word lengths (config validated first). PS5 behaviour remains 13.60-only, unconfirmed on current PS5 firmware. ATRAC9 decoding has only config/error tests (no ATRAC9 encoder fixture).

# libSceCesCs

## change [sceCesJiscsToUcs and sceCesUcsToJiscs]

- [sceCesJiscsToUcs and sceCesUcsToJiscs](../../../core/libs/prx/libSceCesCs/Export.cpp) (libSceCesCs) - device-measured in a payload process: bytes <=`0xFF`, plane 0-2 row/cell `0x21`-`0x7E`, and native sequences convert. Codes >`0xFF` with top byte >=`0x80` hold the first native character (`0x82A0`: U+3042 Shift-JIS / `0x805C0015` EUC-JP; `0xA4A2`: U+3042 EUC-JP / U+FF64 Shift-JIS; `0x8FA2AF`: U+02D8 for the two SS3 profiles; `0x8E21`: `0x805C0014` for all); trailing bytes are ignored and invalid lead bytes return `0x805C0015`. Still throwing: `0x02002422` (U+8CC3 on all profiles, 1-based row 36 cell 34, table for other cells unknown), and `0x10000`, `0x30000`, `0x1002422` (console returns `0x805C0015`). Zero-filled profile sheets give mixed results (`0x80` converts). Null profiles return `0x805C0004` (console faults on `0x8FA2AF`; we return `0x805C0004`).

# libSceContentSearch

## change [Content queries]

- [Content queries](../../../core/libs/prx/libSceContentSearch/Export.cpp) use an empty host library reporting update id 0 (console initial id unverified) and 0 rows; all titles are assumed to have search permission. Incomplete nested filter/sort validation throws on empty nested column sets, null filter values, and unknown enums. Guest heap allocation and platform minimum heap size are unimplemented; init and outer pointer/count/limit checks follow the supplied module. Returning `ErrorInvalidArgument` for unmapped non-null filter/sort arrays or non-null pointers in `sceContentSearchGetTotalContentSize`, `sceContentSearchGetNumOfContent`, and `sceContentSearchGetContentLastUpdateId` is an unmeasured host-safety fallback; in `GetTotalContentSize`/`GetNumOfContent`, the first output is written before copying out the second, so an unwritable second pointer still stores the first value with `ErrorInvalidArgument`.

# libSceDataTransfer

## change [sceDataTransferAbortFgTransfer, sceDataTransferAbortRequest, sceDataTransferAbortSearchPS4, sceDataTransferCheckBgTransferRunning, sceDataTransferGetFgTransferProgress, sceDataTransferGetPrepareFgTransferProgress, sceDataTransferInitialize, sceDataTransferRequestGetAppInfoPS4, sceDataTransferRequestGetSavedataInfoPS4, sceDataTransferRequestGetUsersPS4, sceDataTransferRequestLaunchPS4, sceDataTransferRequestPrepareFgTransfer, sceDataTransferRequestRebootAuthPS4, sceDataTransferRequestSearchPS4, sceDataTransferRequestStartFgTransfer, sceDataTransferRequestTransferTimePS4, sceDataTransferTargetAbortBindSavedata, sceDataTransferTargetAbortDeactivate, sceDataTransferTargetAbortEth0, sceDataTransferTargetAbortGetDeviceInfo, sceDataTransferTargetAbortGetDeviceInfoApplication, sceDataTransferTargetAbortGetTitles, sceDataTransferTargetAbortGetUsers, sceDataTransferTargetAbortLaunch, sceDataTransferTargetAbortPrepareTransfer, sceDataTransferTargetAbortPwrReq, sceDataTransferTargetAbortReboot, sceDataTransferTargetAbortSendSsoNew2Old, sceDataTransferTargetAbortSendSsoOld2New, sceDataTransferTargetAbortTransfer, sceDataTransferTargetAbortTransferSpeed, sceDataTransferTargetEventIsAuthReady, sceDataTransferTargetEventIsIPv6Ready, sceDataTransferTargetEventIsPwrReqReady, sceDataTransferTargetGetFailedUsers, sceDataTransferTargetGetRebootData, sceDataTransferTargetGetTransferProgress, sceDataTransferTargetRequestAbortSearch, sceDataTransferTargetRequestActivate, sceDataTransferTargetRequestAuth, sceDataTransferTargetRequestBindSavedata, sceDataTransferTargetRequestComplete, sceDataTransferTargetRequestCreateRebootData, sceDataTransferTargetRequestDeactivate, sceDataTransferTargetRequestEndTransfer, sceDataTransferTargetRequestGetDeviceInfo, sceDataTransferTargetRequestGetDeviceInfoApplication, sceDataTransferTargetRequestGetTitles, sceDataTransferTargetRequestGetUsers, sceDataTransferTargetRequestLaunch, sceDataTransferTargetRequestPrepareTransfer, sceDataTransferTargetRequestPrepareTransferProgress, sceDataTransferTargetRequestPwrReq, sceDataTransferTargetRequestReboot, sceDataTransferTargetRequestSearch, sceDataTransferTargetRequestSendSsoNew2Old, sceDataTransferTargetRequestSendSsoOld2New, sceDataTransferTargetRequestStartTransfer, sceDataTransferTargetRequestTransferSpeed and sceDataTransferTerminate]

- [60 PS5 3.20 exports (`sceDataTransferAbortFgTransfer` .. `sceDataTransferTerminate`)](../../../core/libs/prx/libSceDataTransfer/Export.cpp) (libSceDataTransfer) - names and NIDs are in the [PS5 3.20 export inventory](https://github.com/DNNDHH/PS5-3.20_Libs/blob/b2a77c09cd71c01f34c790cfbe98c751a13262ad/libSceDataTransfer.c), without credible prototypes. Parameterless declarations exist solely to resolve known imports, not as an ABI claim; callers must not call them until signatures are verified.

# libSceEsvmEngine

## change [sceEsvmEngineDestroyMediaKeySystemAccess, sceEsvmEngineMediaKeySessionClose, sceEsvmEngineMediaKeySessionGenerateRequest, sceEsvmEngineMediaKeySessionGetExpiration, sceEsvmEngineMediaKeySessionGetKeyStatuses, sceEsvmEngineMediaKeySessionGetSessionId, sceEsvmEngineMediaKeySessionLoad, sceEsvmEngineMediaKeySessionRemove, sceEsvmEngineMediaKeySessionSetEventHandler, sceEsvmEngineMediaKeySessionUpdate, sceEsvmEngineMediaKeySystemAccessCreateMediaKeys, sceEsvmEngineMediaKeySystemAccessDestroyMediaKeys, sceEsvmEngineMediaKeySystemAccessGetConfiguration, sceEsvmEngineMediaKeySystemAccessSetCdmStorageConfig, sceEsvmEngineMediaKeysCreateSession, sceEsvmEngineMediaKeysDestroySession, sceEsvmEngineMediaKeysSetEventHandler, sceEsvmEngineMediaKeysSetServerCertificate and sceEsvmEngineRequestMediaKeySystemAccess]

- [19 PS5 3.20 exports (`sceEsvmEngineDestroyMediaKeySystemAccess` .. `sceEsvmEngineRequestMediaKeySystemAccess`)](../../../core/libs/prx/libSceEsvmEngine/Export.cpp) (libSceEsvmEngine) - names and NIDs are in the [PS5 3.20 export inventory](https://github.com/DNNDHH/PS5-3.20_Libs/blob/b2a77c09cd71c01f34c790cfbe98c751a13262ad/libSceEsvmEngine.c), without credible prototypes. Parameterless declarations exist solely to resolve known imports, not as an ABI claim; callers must not call them until signatures are verified.

# libSceFont

## change [sceFontCharactersRefersTextCodes, sceFontTextCodesStepNext and sceFontTextCodesStepBack]

- [sceFontCharactersRefersTextCodes, sceFontTextCodesStepNext and sceFontTextCodesStepBack](../../../core/libs/prx/libSceFont/src/Text.cpp) - [IDA](https://github.com/AlexAltea/orbital/blob/cc408367da6a6ebbf5ac1ac559fefeadc73b2831/tools/ida/db_types.json#L679)/[SP](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/Font/SceFont.Engine.cs#L395-L406); 64B(0/8) [FontTextCodes](https://github.com/SvenGDK/SharpProspero/blob/5dfdb4b0b166e83dc9ae275001afb695f12c7174/src/SharpProspero/Interop/Font/SceFontStructs.cs#L216-L228). PS4 11.00([#4050](https://github.com/boykopovar/AnyPS5/pull/4050#issuecomment-6114504202), no PS5): +0x28/0x30/0x38 cur/start/term; `Refers`!=term, null cluster/synth; steps skip synth, null term/start(assumed end/head). Null->null([Kyty](https://github.com/KytyPS5/KytyPS5/blob/d2413fc2ebd91d1b7234197c2dd5bd48e1cb6a51/src/libs/libFont.cpp#L1696-L1740)); null text throws.

## change [sceFontGraphicsAppendGradientSegment]

- [sceFontGraphicsAppendGradientSegment](../../../core/libs/prx/libSceFont/src/Graphics.cpp) - 4.03 `libSceFont.sprx` disasm (SHA-256 `3c67846f50ed3bf844efdc274f83c7f0043581b607154f3946499b72a3bfa086`: body `0x20500`, span alloc `0x1b5e0`, desc init `0x1bf80`, free `0x1b800`; unmeasured). Errors `0x80460084`/`0x804600a9`/`0x80460020` (null/`-1` grad), `0x80460086` (non-grad), `0x80460002`/`0x804600a0` (arena), no sealed/`frame->object` check; sorted `+8` insert while `node->+4==record->+4 && pos>=node->+0x10`; 0x18B stop `{type,pad,epoch,next,pos,color}`; `0xff` pad+zero tail if `<=0x17`B left; units `frame->+0x18/+0x20`, pos `+0x10/+0x14`; arena `object->+0x10+0xc0` (mutex `+0x40`, stats `+0x48/+0x4c`); node count(22b)`+0`, chain(21b)`+4`, end(21b)`+8`, idx 0 sentinel; writes/splice outside mutex. Inferred: names, pad, arena lifetime. Unknown: consumer, `+4` change, >4.03.

## change [sceFontDefineAttribute, sceFontGetAttribute]

- [sceFontDefineAttribute, sceFontGetAttribute](../../../core/libs/prx/libSceFont/src/Style.cpp), [sceFontGlyphDefineAttribute, sceFontGlyphGetAttribute](../../../core/libs/prx/libSceFont/src/Render.cpp) - `(handle,int attr,int* old/nowAttr)` (+glyph) match [SharpProspero](https://github.com/SvenGDK/SharpProspero/blob/main/src/SharpProspero/Interop/Font/SceFont.Engine.cs), [Orbital](https://github.com/AlexAltea/orbital/blob/master/tools/ida/db_types.json), [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/d2413fc2ebd91d1b7234197c2dd5bd48e1cb6a51/src/libs/libFont.cpp#L1249-L1264) (unconfirmed; no shadPS4). Handle stores int(0); glyph stores bit 0x10/0x11(0x10, else `SCE_FONT_ERROR_INVALID_PARAMETER`); get ignores `attr`. Null/invalid checks diverge from [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/d2413fc2ebd91d1b7234197c2dd5bd48e1cb6a51/src/libs/libFont.cpp#L1251-L1265) (`OK`+0). PPSA01325 sets glyph 0x11; no render/shape effect.

# libSceFreeTypeOptOl

## change [FT_Get_PFR_Advance, FT_Get_PFR_Kerning and FT_Get_PFR_Metrics]

- [FT_Get_PFR_Advance, FT_Get_PFR_Kerning and FT_Get_PFR_Metrics](../../../core/libs/prx/libSceFreeTypeOptOl/Export.cpp) use LP64 adapter and [FreeType 2.13.3](https://github.com/freetype/freetype/blob/42608f77f20749dd6ddc9e0536788eaad70ea4b5/src/base/ftpfr.c) via [Pfr.cpp](../../../core/libs/prx/libSceFreeType/src/Pfr.cpp). Null face returns Invalid_Argument(6) untouched, matching [FreeType 2.4.11](https://github.com/freetype/freetype/blob/VER-2-4-11/src/base/ftpfr.c) and [BunnyFl fw 13.42 Unicorn](https://github.com/boykopovar/AnyPS5/pull/9646#issuecomment-6405909027) (2.4.11 Ol disasm; NIDs in OptOl/Ol under libSceFreeType; no real-face runs or full fw). Exported only in OptOl (Ol missing). Sony ABI/layouts/versions/nonnull parity unverified; 64-bit guest storage on Windows too. Nonnull follows 2.13.3 (invalid idx writes 0; non-PFR advance untouched, metrics writes fallbacks before error).

# libSceHmd2

## change [libSceHmd2 reprojection session]

- [libSceHmd2 reprojection session](../../../core/libs/prx/libSceHmd2/Export.cpp) per [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/main/src/libs/libHmd2.cpp) (`static_assert` 16B/8-align buf-start state under mutex; no stage). `QueryBufferSizeAlign`: 0x100828/0x100 unset(13.60,vs 16B), 0x1103000+n*0x1000000/0x1000 n=1..5(13.40,args ignored). `SetAllowPositionalReprojection`: 0..5 pre-init(PPSA23479/PPSA05666=2,PPSA01317 unset; session unmeasured), 0x8A720009 on 6..8/0xFF/0xFFFFFFFF(13.40). `Initialize`: skips 13.40 AppInfo 0x400(0x8A72001F,#8645), 0x8A720008 null, aux throws, 3000us, 0x8A72000C re-init. `SetTiming`: 0x8A72000B, 0x8A720009 !2000..7000us(13.40), rounds 100us(`sceVideoOutSetDisplayEventPosition` unimpl). `EnableVrMode`: 0x8A72000B, 0x80290001 !=0x2000C(89/91Hz)/0x2000F(119/88Hz), else 0x80290018([errno.h](https://github.com/KytyPS5/KytyPS5/blob/main/src/libs/errno.h)). `Terminate`: clears(unmeasured; setting kept).

## change [Hmd2BeforeInitializeZero]

- [Hmd2BeforeInitializeZero](../../../core/libs/prx/libSceHmd2/Export.cpp) (libSceHmd2) - before `sceHmd2Initialize`, fw 13.60 payload measurements return 0 (writing nothing) for both all-null args and all-zeroed-16 KiB-buffer args across 45 exports (`vDgpdS2N9Rw`/`sceHmd2InternalRegisterVibrationCallback` .. `IsC4R4bZFVU`, including DFU, crash report, IPD offset, relocalizer, tracking, LED detector, image, gen data, SLAM trace, gaze, shared memory, and VR2D/MiniApp/CommonDlg queries). Game process before init is assumed identical; arg semantics and post-init behaviour are unmeasured, so handlers throw once `sceHmd2Initialize` succeeds.

## change [Hmd2CheckFirstThenError0004]

- [Hmd2CheckFirstThenError0004](../../../core/libs/prx/libSceHmd2/Export.cpp) (libSceHmd2) - before `sceHmd2Initialize` (probe `hmd2n`, payload process, fw 13.60, each arg nulled individually), 20 exports (`9WeZr4kYfzk`/`sceHmd2InternalGetFwVersion` .. `638Tmg5y6U4`, covering optical/camera see-through, FTS/SLAM/DFU/relocalizer/home calibration, and gaze status) return `0x8a720008` when arg 0 is null and `0x8a720004` when all args point to a zeroed 16 KiB buffer, writing nothing. Arg meanings and post-init behaviour are unmeasured; handlers throw once `sceHmd2Initialize` succeeds.

# libSceHttp

## change [libSceHttp]

- [libSceHttp](../../../core/libs/prx/libSceHttp/Export.cpp) - no request reaches the network: `sceHttpSetResponseHeaderMaxSize`, `sceHttpSetRecvBlockSize` (`(int id, u32 blockSize)`, [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/network/http.h)), `sceHttpRedirectCacheFlush`, `sceHttpsUnloadCert`/`sceHttpsLoadCert`, `sceHttpSetRedirectCallback`, `sceHttpSetCookieRecvCallback`, `sceHttpSetAuthInfoCallback`, and `sceHttpsSetSslVersion` are no-ops/uninvoked; `sceHttpsGetSslError` reports error/detail 0. `sceHttpsGetCaList`/`sceHttpsFreeCaList` return 0 with empty list (`certs` null, `certsNum` 0) or `ERROR_INVALID_VALUE` on null list (skipping shadPS4's `http_ctx_id`/init checks; `FreeCaList` also nulls `certs`). Signatures and `{ void* certs; int certsNum; }` follow [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/main/src/core/libraries/network/http.h) (PS4, unconfirmed on PS5).

## change [sceHttpUriParse]

- [sceHttpUriParse](../../../core/libs/prx/libSceHttp/src/Uri.cpp) (libSceHttp) - follows PS5 fw 13.60 payload for every ASCII byte in scheme, userinfo, and hostname, plus `[A-Za-z0-9+-.]` at scheme start (digit ends it; other leading bytes treated as inside scheme). Userinfo ends at first `@` after RFC 3986 userinfo bytes; hostname allows `[A-Za-z0-9-_.]` without leading `.`, and the first byte outside starts the path instead of failing (`http://bad host/` -> host `bad`, path ` host/`). Port is up to 5 digits ending at `/` or end, max 65536 (reads 0); empty port defaults to 443 (`https*`), 80 (`http*`, case-insensitive), else 0. Path is swept like `sceHttpUriSweepPath` while pool reserves unswept size. Undersized pool returns `ERROR_OUT_OF_MEMORY` leaving `require` and pool untouched (console partially writes pool).

# libSceIme

## change [sceImeOpen, sceImeUpdate, sceImeSetText, sceImeSetCaret, sceImeClose]

- [sceImeOpen, sceImeUpdate, sceImeSetText, sceImeSetCaret, sceImeClose](../../../core/libs/prx/libSceIme/Export.cpp) - events(0 Open,1 Text,2 Caret,4 Close,5 Enter), caret(1 L,2 R), errors, payload per [shadPS4](https://github.com/shadps4-emu/shadPS4/blob/b72489d8c24e63fc56a29fe2c90120ab768a210d/src/core/libraries/ime/ime.cpp)(no PS5). `Update` needs `Open` handler/arg; text events set `str` in `work`(=terminated `inputTextBuffer`,`caret_index`,UTF-16 pos/delta,0 Enter/Close); 1 UpdateText/edit, 1 UpdateCaret/UTF-16 move. Enter->newline(multiline)/PressEnter; Esc->PressClose. Cuts `maxTextLength`(1..2048) keeping surrogates; BasicLatin/URL/mail/num unfiltered. `Open` checks user/type/enter/method/panelMask/`maxTextLength`/buf/pos(<1920x1080,3840x2160>2K)/align/prio/4B-`work`/cb/init(skips SDK masks,overlap; text only in `work`). `SetText`/`SetCaret`(`0x80bc0030` OOB) emit nothing; `SetTextGeometry` takes any open geom.

## change [sceImeOpen text filter]

- [sceImeOpen text filter](../../../core/libs/prx/libSceIme/Export.cpp) runs on host edits (typing, backspace, delete, multiline Enter) per [KytyPS5](https://github.com/KytyPS5/KytyPS5/blob/1211958b4eb38256cc0c5c7ba92d668820c05389/src/libs/ime.cpp) (PS5-unconfirmed). Host queues under IME mutex; `sceImeUpdate` runs `ImeTextFilter`/`ImeExtKeyboardFilter` (`APS5_VABI`, like `EventHandler`) unlocked on title thread to avoid deadlock, serializing via session id+revision. Filter gets text, `maxTextLength` UTF-16 buf, len; nonzero, overflow, unpaired surrogate, or non-multiline newline keeps edit unfiltered. Accepted output updates text, clamps/normalizes caret past surrogates, emits minimal-delta UpdateText (skipped if unchanged). Caret moves, non-multiline PressEnter, PressClose, `SetText`, `SetCaret` bypass filter; `SetText`/`SetCaret` mid-filter bump revision, dropping edit. Ext keyboard filter never called (zeroed key).

# libSceJxr

## change [PixelFormatLookup]

- [PixelFormatLookup](../../../core/libs/prx/libSceJxr/src/PixelFormatLookup.cpp) uses pinned jxrlib via C bridge and 56-byte LP64 guest record. [BunnyFl fw 13.42 PC comparisons](https://github.com/boykopovar/AnyPS5/pull/9954#issuecomment-6406356097): 68 table entries and 802 cases x3 match except module zeroes padding bytes 28..31 on all 309 successes (bridge now zeroes 28..31 explicitly). Single-firmware attributed evidence only; module/lib versions, other fw, and valid-mode null args remain unverified. Modes 0/1 do forward GUID/reverse TIFF lookup; other modes/misses preserve record. Valid-mode null records/GUIDs are unsupported; success borrows static read-only GUID while loaded. Closes when remaining Sony ABI, version/config, and invalid-arg behavior are verified by attributable measurements or disassembly.

# libSceLibreSsl

## change [BN_new, BN_free, BN_CTX_new, BN_CTX_free, BN_CTX_start, BN_CTX_end, BN_bin2bn, BN_bn2bin, BN_hex2bn, BN_num_bits, BN_set_word, BN_add_word, BN_sub_word, BN_mul and BN_mod_inverse]

- [BN_new, BN_free, BN_CTX_new, BN_CTX_free, BN_CTX_start, BN_CTX_end, BN_bin2bn, BN_bn2bin, BN_hex2bn, BN_num_bits, BN_set_word, BN_add_word, BN_sub_word, BN_mul and BN_mod_inverse](../../../core/libs/prx/libSceLibreSsl/src/Bignum.cpp) (libSceLibreSsl) - console behaviour unverified; `BIGNUM` layout (24B, 64-bit limbs, `top`, `dmax`, `neg`, `flags`) and semantics follow LibreSSL 2.x. `BN_CTX` only counts `BN_CTX_start`/`BN_CTX_end` (`BN_CTX_get` unimplemented; `BN_mul`/`BN_mod_inverse` ignore it). `BN_mod_inverse` uses magnitude of negative modulus and returns 0 for modulus 1 (uncompared with LibreSSL). `BN_hex2bn` clears/allocates on non-empty hex without digits, keeping negative zero per LibreSSL 2.5.4. Null `BIGNUM` pointers throw (unlike LibreSSL). `BN_print` emits one hex digit per `BIO_write` call (`bn_print.c`).

# libSceMbus

## change [sceMbus event subscriptions]

- [sceMbus event subscriptions](../../../core/libs/prx/libSceMbus/PendingExports.cpp): subscription carries two 32-bit and two 64-bit fields with unresolved service meanings. EventCreate uses 80-byte queue records and copies a 16-byte callback table (event dispatch fields needed; overflow callback takes event handle + user data, event callback adds queued event pointer). EventCallbackFuncsInit has no defined return value and clears memory; treating arg 2 as byte count is unverified against clamping in [PR #9137](https://github.com/boykopovar/AnyPS5/pull/9137) (no fw/offset/instructions recorded). 11.00 wrapper ignores null callback pointer before clearing (host matches; unmeasured on console, 13.42 unverified). Eight dump/BGM wrappers have confirmed 0-arg ABIs. EventBusStatusChangeUnsubscribe ignores arg 1 (type unresolved) and takes a 32-bit arg 2.

# libSceNKWebKit

## change [WebCore layout helpers]

- [WebCore layout helpers](../../../core/libs/prx/libSceNKWebKit/src/WebCoreLayout.cpp) - `valueForLength`, `floatValueForLength`, `GridPosition::setSpanPosition`, `GridPosition::setExplicitPosition`, `GridPosition::setMaxPositionForTesting`, `unitedBoundingBoxes` follow [LengthFunctions.cpp](https://github.com/neonmodder123/PS5-WebKit-11.x/blob/80f77e94c63ee7202eec3d4b5b1a021dd89011ba/Source/WebCore/platform/LengthFunctions.cpp), [GridPosition.cpp](https://github.com/neonmodder123/PS5-WebKit-11.x/blob/80f77e94c63ee7202eec3d4b5b1a021dd89011ba/Source/WebCore/rendering/style/GridPosition.cpp), [FloatQuad.cpp](https://github.com/neonmodder123/PS5-WebKit-11.x/blob/80f77e94c63ee7202eec3d4b5b1a021dd89011ba/Source/WebCore/platform/graphics/FloatQuad.cpp). Calculated throws (`CalculationValue` unimpl.); span clamp `[-max, max]` (parser enforces positive); percent uses WebKit float math with truncating `LayoutUnit(float)`. Unmeasured on console.
