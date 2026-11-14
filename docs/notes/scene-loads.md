# Scene loads and the scene request — Crash Bandicoot: Warped (`SCUS_942.44`)

Read from the executable in Ghidra and observed in the psx-crash port (store watches, RAM dumps at the store).

## Request to load

- The frame loop `NativeFrameCoreLoop` (0x8001166C) reads the request word 0x80060AC0 at 0x80011938. When it is not
  -1 it calls the level load 0x80014BC8 at 0x800119E4 with a0 = 0x80061030 and a1 = the request.
- The level load writes -1 to the request, a1 to the scene id 0x80060ABC and to a0 + 4, then runs each subsystem's
  init from its loop at 0x800150C0 (return address 0x800150D8).
- Subsystems: table at 0x8005BC98, 22 entries of 0x1C bytes, each starting with a four-character name. The handler at
  entry + 0xC is the init.
- Slot 6 is `LDAT`; its init is 0x80021084. It compares the level id of the loaded NSD, `*(*(0x80061044) + 8)`, with
  the scene id and, when they differ, branches to 0x800210FC, a `j .` to itself. A request for a scene whose NSD is
  not the one loaded therefore never returns.
- The scene table `FrameSceneTable` (0x8006C4D8) has no row for scenes 0 and 1 (both zero), so no NSD loads for them.
  A request for scene 0 after a level is loaded ends in the `LDAT` trap.

## GOOL writes

- The GOOL interpreter dispatches at 0x80038E80 through the opcode table 0x8005D120, reached through scratchpad
  0x1F80005C. Operands resolve at 0x80038C94: ref 0xE1F pops the object's stack, 0x8xx with bit 9 clear is the
  literal `sext9(ref) << 8`, 0xBE0 is 0.
- Object fields: +0xBC stack pointer, +0xC0 code pointer, +0xC4 frame pointer.
- Op 0x1C calls 0x80035D08(object, word). Its sub-op `(word >> 20) & 0xF == 0xC` stores `value >> 8` to the scene
  request at 0x80036798: GOOL code is what asks for a level.
- Op 0x1F pushes `(val(B) + *(0x1F800050)) mod val(A)`. Scratchpad 0x50 is a copy of the frame counter 0x80069034,
  which `NativeFrameGpuUpdate` advances while state + 0x84 has bit 0x1000.
- Op 0x20 pushes global `G[val(A) >> 8]` from the table at `*(0x1F800058)` (0x80068EF8); op 0x21 stores one.

## Scene 0x3A after a warp

In the port, a direct warp to scene 0x3A (an Aku Aku screen) sometimes ends in the `LDAT` trap exactly 541 frames
after the load, on two machines. In each case the request store is GOOL op 0x1C sub-op 0xC with word 0x1CC4DE1F at
0x800BB6B0 (object 0x800ACAB4, code header 0x800B8E0C) and pops a value of 0 from a frame at 0x800ACBFC whose words are
0xFFFF, 0x800B8EB8, 0x108, 0. Runs that do not trip never make that store within 30 s. Booting with no input stays
on scene 0x3C, and New Game goes 0x3C, 0x0B, 0x3C, 0x28, 0x02 without reaching 0x3A, so how the game normally enters
0x3A, and which event delivers the 0, is still open.
