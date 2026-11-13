# Our names (`SCUS_942.44`)

The names in this repository's symbol map (`config/symbols/scus_942.44.txt`) are the psx-crash port's, taken from the
file and key that holds each address there (`titles/crash3/executable.json` paths, or `entry/facts.cpp` initializers).
They describe the port's use of an address, not yet the guest's own meaning; a better name replaces one here and in
the map together, and the port adopts it when its constants are generated (psx-crash
`docs/plans/0001-addresses-from-the-decomp-symbol-map.md`).

## Names added to the map

| address | added as | kind | port source |
|---|---|---|---|
| `0x80011628` | `NativeBootCMain` | func | `runtime.native_boot.c_main` |
| `0x8001166C` | `NativeFrameCoreLoop` | func | `runtime.native_frame.core_loop` |
| `0x80011F04` | `NativeFrameTransition` | func | `runtime.native_frame.transition` |
| `0x80012CBC` | `LoaderPollService` | func | `entry/facts.cpp kLoaderPoll.service` |
| `0x800154AC` | `NativeBootInit` | func | `runtime.native_boot.init` |
| `0x80016470` | `SceneLoadEntry` | func | `runtime.native_frame.scene.load_entry` |
| `0x80016634` | `NativeFrameGpuUpdate` | func | `runtime.native_frame.gpu_update` |
| `0x80017AE8` | `NativeBootCallbackInitialize` | func | `runtime.native_boot.callback_initialize` |
| `0x8003AE70` | `RootCounterIncrement` | func | `runtime.native_frame.root_counter_increment` |
| `0x80045A1C` | `NativeBootCdControl` | func | `runtime.native_boot.cd_control` |
| `0x80045B54` | `NativeBootCdControlF` | func | `runtime.native_boot.cd_control_f` |
| `0x80046574` | `NativeBootCdSync` | func | `runtime.native_boot.cd_sync` |
| `0x80047188` | `NativeBootCdInitialize` | func | `runtime.native_boot.cd_initialize` |
| `0x80047E40` | `NativeBootCdRead` | func | `runtime.native_boot.cd_read` |
| `0x80047F44` | `NativeBootCdReadSync` | func | `runtime.native_boot.cd_read_sync` |
| `0x800489F8` | `Crt0Entry` | func | `entry/facts.cpp kCrt0.entry` |
| `0x80048AA0` | `GameMainEntry` | func | `runtime.game_main_entry` |
| `0x80048B98` | `CallbackBootChangeClearPad` | func | `entry/facts.cpp kCallbackBoot.changeClearPad` |
| `0x80048BE8` | `CallbackBootOpenEvent` | func | `entry/facts.cpp kCallbackBoot.openEvent` |
| `0x80048C18` | `CallbackBootCloseEvent` | func | `entry/facts.cpp kCallbackBoot.closeEvent` |
| `0x80048C38` | `FirstSyscallEntry` | func | `runtime.first_syscall_entry` |
| `0x80048C48` | `CallbackBootExitCritical` | func | `entry/facts.cpp kCallbackBoot.exitCritical` |
| `0x80048CD8` | `SetRootCounter` | func | `runtime.native_frame.set_root_counter` |
| `0x80048DAC` | `StartRootCounter` | func | `runtime.native_frame.start_root_counter` |
| `0x80048DDC` | `StopRootCounter` | func | `runtime.native_frame.stop_root_counter` |
| `0x8004B2A8` | `Vsync` | func | `runtime.vsync` |
| `0x8005B6A0` | `CallbackBootInitializePad` | func | `entry/facts.cpp kCallbackBoot.initializePad` |
| `0x8005B6F8` | `CallbackBootStartPad` | func | `entry/facts.cpp kCallbackBoot.startPad` |
| `0x8005DE4C` | `LibcdInitLastSyncCallback` | data | `entry/facts.cpp kLibcdInit.lastSyncCallback` |
| `0x8005DE50` | `LibcdInitLastReadyCallback` | data | `entry/facts.cpp kLibcdInit.lastReadyCallback` |
| `0x8005DE5C` | `LibcdInitLastStatus` | data | `entry/facts.cpp kLibcdInit.lastStatus` |
| `0x8005DE60` | `LibcdInitLastResult` | data | `entry/facts.cpp kLibcdInit.lastResult` |
| `0x8005DE6C` | `LibcdInitLastCommand` | data | `entry/facts.cpp kLibcdInit.lastCommand` |
| `0x8005DE6D` | `LibcdInitPendingCommand` | data | `entry/facts.cpp kLibcdInit.pendingCommand` |
| `0x8005E12C` | `LibcdInitSyncStatus` | data | `entry/facts.cpp kLibcdInit.syncStatus` |
| `0x8005F384` | `VsyncQueryCounter` | data | `runtime.vsync.query_counter` |
| `0x800608A4` | `NativeBootUseCdAddress` | data | `runtime.native_boot.use_cd_address` |
| `0x800608E4` | `NativeFrameDone` | data | `runtime.native_frame.done.address` |
| `0x800608E8` | `LimiterVblankCounter` | data | `entry/facts.cpp kLimiter.vblankCounter` |
| `0x80060ABC` | `SceneIdAddress` | data | `runtime.native_frame.scene.id_address` |
| `0x80060AC0` | `SceneRequestAddress` | data | `runtime.native_frame.scene.request_address` |
| `0x80061A7C` | `LimiterDisplayedFrame` | data | `entry/facts.cpp kLimiter.displayedFrame` |
| `0x800691FC` | `LoaderPollLoadingFrames` | data | `entry/facts.cpp kLoaderPoll.loadingFrames` |
| `0x8006C4D8` | `FrameSceneTable` | data | `entry/facts.cpp kFrame.sceneTable` |

## Code sites

Addresses inside a function the port stops at, returns to or patches; not symbols.

| address | where | port source |
|---|---|---|
| `0x800116D8` | `NativeFrameCoreLoop + 0x6C` | `runtime.native_frame.iteration.entry` |
| `0x80011F04` | `NativeFrameCoreLoop + 0x898` | `runtime.native_frame.done.load_entry` |
| `0x80012F08` | unplaced | `entry/facts.cpp kLoaderPoll.afterService` |
| `0x80016AA8` | `NativeFrameGpuUpdate + 0x474` | `runtime.native_frame.first_vsync` |
| `0x80016AB0` | `NativeFrameGpuUpdate + 0x47C` | `runtime.native_frame.after_first_vsync` |
| `0x80016AD4` | `NativeFrameGpuUpdate + 0x4A0` | `runtime.native_frame.second_vsync` |
| `0x80016ADC` | `NativeFrameGpuUpdate + 0x4A8` | `runtime.native_frame.after_vsync` |
