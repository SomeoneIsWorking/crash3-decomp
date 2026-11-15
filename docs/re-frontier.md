# RE frontier — Crash Bandicoot: Warped (`SCUS_942.44`)

The ordered evidence chain the psx-crash port follows toward faithful behaviour, run with re-harness
`re_frontier.py` from this repository's root (from the port: `RE_FRONTIER_ROADMAP=external/crash3-decomp/docs/re-frontier.md`).
`where` and evidence paths (`game/...`, `titles/...`, `tools/...`, `docs/issues/...`) are psx-crash paths. A dependency
on a step kept in another title's roadmap is named in `notes` as `Cross-title dep`, not in `deps`. The trilogy's
roadmaps are in `crash-bandicoot-decomp` (Crash 1, and SHARED-01, the cross-title ownership step), `crash2-decomp` and
`crash3-decomp`.

The execution-method migration is ordered in psx-crash `docs/migration.md`. Static-path observations below are frozen
evidence for the frontier that Lightrec must reproduce; they are not instructions to regenerate, build, or run that
path, and they do not make it a product fallback.

Statuses: ✅ `re-verified` · 🟡 `re-partial` · 🔬 `in-progress` · ⬜ `todo` · ⏸ blocked upstream ·
➖ `skip-by-design` · ⚠️ `hack`.

## Target identity

### CRASH3-01 — select and measure the Crash Bandicoot 3 executable
- status: re-verified
- deps:
- evidence: The real USA disc's `SYSTEM.CNF` selects `SCUS_942.44`, not its bundled `DRAGON/SPYRO.EXE`. The extracted 333,824-byte PS-X EXE has SHA-1 `ffa9252384eb16ce5ae14061bb87c86701ab4893`, SHA-256 `1b93cc563c90966e1d79b750f7d19bc1662859cceeb991775e9e9a22656d71d1`, entry `0x800489F8`, text `[0x80010000,0x80061000)`, gp `0`, and stack `0x801FFFF0`; the North America marker is present. The executable gate matches 11/11 tracked facts. The separately extracted 393,216-byte `DRAGON/SPYRO.EXE` disagrees on name, size, both hashes, entry, and text size.
- where: `titles/crash3/executable.json`, `tools/verify_executable.py`, gitignored `scratch/bin/crash3/SCUS_942.44`
- gap: This proves executable identity and the disc's configured boot selection only; it does not authenticate the whole disc, execute the game, or make bundled demo content part of Crash 3.
- notes: The title codeword is `SCUS-94244`. No Crash 1, Crash 2, or Spyro address is inherited.

### CRASH3-02 — provision the selected disc and verify the first crt0 call
- status: re-verified
- deps: CRASH3-01
- evidence: The shared serial-aware provisioner publishes only after SYSTEM.CNF boots SCUS_942.44 and the executable matches 11/11 facts. Its 10-case suite includes three-way environment isolation, wrong-Crash-title refusal, a positive Crash 3 fixture carrying DRAGON/SPYRO.EXE, and refusal when that bundled path is made the boot target. Against clean pinned psxport 57a17a14, crt0_extract and the independent Mednafen CPU oracle agree 6/6 at executed jal ordinal 1, step 71,790, target 0x800112B8: gp 0x80060878, a0 0x8006EA7C, sp 0x801FFFF8, and heap size 0x00190580. The oracle's 39-case fixture passes independently.
- where: `tools/provision_title.py`, CMake `crash3_oracle_boot_check`, gitignored `scratch/bin/crash3/SCUS_942.44`
- gap: This proves executable selection and the first real crt0 call boundary. Later boundary evidence is separately tracked by CRASH3-04; this step alone proves no BIOS continuation, hardware, native boot, or gameplay.
- notes: The first structural call target is not an A(39h) thunk.

### CRASH3-03 — own measured executable facts through the derived runtime
- status: re-verified
- deps: CRASH3-02
- evidence: Against clean pinned psxport 57a17a14, Crash3Runtime directly inherits the title-agnostic BoundaryRuntime, which in turn inherits GameRuntime; both legacy views and unmeasured runtime products remain null. Its GuestProgramImage matches 15/15 fields derived from real SCUS_942.44 and the shipping crt0 decoder, including gameMainEntry 0x80048AA0 verified as oracle call three; an altered global pointer produces one named disagreement. Authoritative Clang CTest passes 5/5, including all three runtime inheritance tests, the 10-case provisioning contract, and format/size/clang-tidy policy.
- where: `titles/crash3/entry/facts.cpp`, `titles/crash3/entry/runtime.*` over `game/runtime/runtime.*`; `tests/runtime.cpp`
- gap: The typed runtime owns measured executable facts and game main but still refuses native boot; CRASH3-04 preserves the later boundary evidence, and CRASH3-JIT-01 owns future product execution.
- notes: The refusing `BoundaryRuntime` is deleted; Crash 3 runs on `crash::Runtime` (CRASH3-JIT-01).

### CRASH3-04 — recorded resident boundary evidence
- status: re-partial
- deps: CRASH3-03
- evidence: Real SCUS_942.44 emits 986 candidates from 297 static seeds with zero overlays under clean psxport `8611d756` (emitter 2026-08-24.2). Generated execution and the independent Mednafen CPU agree 34/34 at each of eight calls: 0x800112B8, 0x80011628, game main 0x80048AA0, 0x800154AC, 0x8004BFBC, 0x8004F37C, 0x8004F914, and 0x80048C38. Call eight is the measured addiu-a0-1/syscall-0 wrapper. At step 75963 the oracle records Cause `0x20` and EPC `0x80048C3C`; the shipping HLE retains that record, returns prior IRQ 1, disables delivery, and both CPUs pop the Status mode stack once to resume at `0x80048C40`. Their first post-return state then agrees 34/34 at dynamic BIOS dispatch PC `0xB0`. The 16/16 suite includes altered-frontier, out-of-text seed, 7/8 short-window, repeated-target, altered-register, wrong-syscall-target, and wrong-selector refusals.
- where: frozen evidence only; the static translator, seeds, and boundary tests were deleted by the execution migration and are not restored. The typed runtime facts it bound live in `titles/crash3/executable.json` and `titles/crash3/entry/facts.cpp`.
- gap: Equality is proven from entry through the first post-syscall B0 dispatch boundary. The B0 HLE call itself, later BIOS/hardware boot, frame output, and the remaining 978 emitted bodies are unverified.
- notes: Static discovery counts are not execution provenance; only nine addresses are execution-proven. Preserve this measurement without extending the static path. The crt0 decoder's structural libcInit slot 0x800112B8 is not an A(39h) thunk in SCUS_942.44; do not apply the A0 modeled-return path to it.

## Preserved execution and ownership evidence

### CRASH3-VSYNC — identify and bound guest VSync
- status: re-partial
- deps: CRASH3-01
- evidence: Verified SCUS_942.44 contains canonical libetc VSync at `[0x8004B2A8,0x8004B3F0)`. The tracked body SHA-256 is `9de0bd51589e40ceb1c85119f26f3a0ab8ffd90e8435df340c529444e8a52e80`. The executable gate retains altered entry/body controls, and the exact-pinned asset-free contract test proves the production platform owner returns a typed `FrameBoundary`.
- where: `titles/crash3/executable.json`, `game/frame/frame_driver.*`, `crash::Runtime::registerOverrides` (`StockVSync`)
- gap: Reached live through Lightrec at both GpuUpdate sites (CRASH3-FRAME).
- notes: The address is a Crash 3 fact; neither Crash 1 nor Crash 2 addresses are reusable.

### CRASH3-FRAME — recover one complete native-owned frame step
- status: re-verified
- deps: CRASH3-04, CRASH3-VSYNC
- evidence: CoreLoop `[0x8001166C,0x80011F68)` resumes at `0x800116D8`; GpuUpdate `[0x80016634,0x80016CE8)` is called at `0x80011EFC` and returns to `0x80011F04`, where the transition override ends the host frame (Crash 1's design; issue 0028). Waits at `0x80016AA8`/`0x80016AD4` run only while fewer than two vblank callbacks (`0x80017AD0`, `gp+0x70`) have passed since the last stamp (`0x80016A8C`), so the two frames after a load (`0x80061054` non-zero keeps the displayed record's stamp) commit the fields up to their VBlank. Init programs CNT2 with target `0x1010`. The level loader's wait `FUN_80012EAC` calls the page-load service `FUN_80012CBC(-1)` (returns to `0x80012F08`) and draws a loading frame through GpuUpdate(1) only once `gp+0x70` differs from `displayed->0x1C`, while `0x800691FC` (loading frame count, -1 when done) is non-negative; the VBlank callback only runs between host frames, so that poll ends the host frame (`FrameProgram::loaderPoll`, one field when it was waiting).
- where: `titles/crash3/executable.json` `runtime.native_frame`, `titles/crash3/entry/facts.cpp`, `game/frame/frame_driver.*`
- gap: none for the frame step.
- notes: Body hashes in `executable.json` are checked by `tools/verify_executable.py`.

### CRASH3-WORLD — the world renderer and its screen test
- status: re-partial
- deps: CRASH3-FRAME
- evidence: `Gfx_DrawWorlds` 0x80040ED8, its default steps, three link tails and five scene modes are read (`docs/notes/world-renderer.md`); the ten packed-bound screen tests are placed (`docs/notes/screen-culls.md`). The port's host renderer is byte-identical to the guest at 4:3 in default, sway and glow under the override differential; underwater differs only in each polygon's last corner colour, where the guest under Lightrec stores a stale value instead of the `mfc2` in the branch delay slot (psx-crash issue 0030).
- where: `game/render/world_*`, `titles/crash3/render/world_*`
- gap: Pulse and the own-slot tail are reached at no level entry; the scene effects (0x80043310, 0x80043FFC, 0x800452A0) and object drawers still cull at x = 512; 0x8003CB24's caller is not traced.
- notes: Cross-title dep: crash2-decomp's world renderer; one host implementation serves both titles.

### CRASH3-REPLAY — the draw section and its inputs for in-betweens
- status: re-partial
- deps: CRASH3-FRAME, CRASH3-WORLD
- evidence: `docs/notes/frame-loop.md`: the picture half from the `Gfx_UpdateView` call 0x80011A4C to `NativeFrameGpuUpdate` 0x80016634, `Obj_UpdateAll` 0x8001D16C, the sample point `Sys_BeforeDrawObjects` 0x80011FC0, the object pool, camera, shake and fade. Replayed at t = 1 after the object walk, 3,945 of 3,950 frames of `warp 5` match the real ordering table word for word; the other five are each the first frame after `Sys_LoadCheckpoint` 0x80030388 (frame counter reset to 1), which the port declares a cut.
- where: `titles/crash3/entry/facts.cpp` (`kReplay`), `titles/crash3/frame/camera_cuts.*`, `game/replay/*`
- gap: The vertex-animated pose fetch and morph drawer; `Cam_Update`'s own cuts.
- notes: Cross-title dep: crash2-decomp's frame loop; the replay owner is shared.

## Runtime execution migration

### CRASH3-JIT-01 — migrate Crash 3 after Crash 2 is complete
- status: re-partial
- deps: CRASH3-02, CRASH3-03, CRASH3-VSYNC
- evidence: Boots on the shared Crash owners to the title menu; `warp 5` and `warp 3` run the N. Gin and Dingodile boss levels; issue 0028 resolved. Crt0 and C main's steps (static constructors `0x80048AA0`, CD flag `0x800608A4`, Init `0x800154AC`), libcd init `0x80047188` (Crash 2's body), stock libcd entries, callback init `0x80017AE8` (Crash 2's layout), scene table `0x8006C4D8` (rows 2..7 and 0xA..0x14 populated after boot), code-module arena `0x0006EA78..0x00200000`. The GPU watchdog check `0x8004EB80` differs from Crash 1 and 2 and runs as guest code; libpad `PadInitDirect` reads the pad through the title's SIO code.
- where: `titles/crash3/` plus psxport executor
- gap: Reproduce Crash 3's recorded resident boundary through Lightrec, recover its title-owned frame/services, and pass representative gameplay.
- notes: Preserve `SYSTEM.CNF` selection and reject `DRAGON/SPYRO.EXE` as the Crash 3 product image. Cross-title dep: CRASH2-JIT-01.
