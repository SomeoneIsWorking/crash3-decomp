# Frame loop and draw replay — Crash Bandicoot: Warped (`SCUS_942.44`)

Read from the executable in Ghidra and its disassembly; checked in the psx-crash port's draw replay (each real
frame's section run again at t = 1 and its ordering table compared word for word with the real frame's).

## `NativeFrameCoreLoop` (0x8001166C), one iteration

1. The pause menu and the scene request (0x80060AC0; see `scene-loads.md`). A request of -2 reloads the checkpoint
   through `Sys_LoadCheckpoint` after the level load.
2. Entry paging `LoaderPollService(-1)` (0x80012CBC).
3. When the frame flags 0x80068F1C lack 0x40000000: `Cam_Update` 0x80028A78 (only while the pause word 0x44 is 0),
   then `Gfx_UpdateView` 0x80018A54, called at 0x80011A4C. With 0x40000000 set the ordering table is reset
   (`Gfx_ResetOt` 0x8003AC44 on build buffer + 0x20, 0x800 slots) and `Obj_UpdateAll` runs here instead, before the
   world.
4. `SetRotMatrix` 0x8004F674 on the view, then the worlds when flags bit 0, the zone's world count
   (`**(*(0x80065D5C) + 0x10)`) and 0x80065DD4 are all set: the zone's world records are copied to scratchpad
   0x1F800280 (eight words each), and `Gfx_DrawWorlds` 0x80040ED8 is called at 0x80011C04 with the sort list
   `*(0x80060AA4)`, the build buffer, the scene 0x800676B0 and the frame counter. Otherwise the table is reset.
5. Scene effects while flags & 0x20000001: 0x80043050, 0x80043310, the water grid 0x80043FFC, 0x800452A0, 0x8005D370,
   0x800435DC, each gated by bits of 0x80067A28.
6. `Obj_UpdateAll` 0x8001D16C (its call at 0x80011D88 follows a branch inside one block, so a stretch that must stop
   there stops at the function's entry). It hands `Obj_Update` 0x8001D674 to `Obj_ForEachList` 0x80019A38; an object's
   logic runs only while flags bit 8 is set.
7. `Sys_BeforeDrawObjects` 0x80011FC0, an empty function, at 0x80011D90.
8. Scratchpad 0x1F800040..0x5F saved; `Gfx_DrawObjects` 0x8003F3AC on the build buffer's queue (+0x2090) at
   0x80011D98; then, when the loading-frame count 0x800691FC is -1, 0x8001CD80 for 0x800690C4 with the scene in
   0x80060A98 and for 0x800691E0 with 0x80060A9C (each draws the queue again through that scene into a viewport);
   both scene words are then zeroed. Scratchpad restored.
9. The list at build buffer + 0x209C, each entry's object drawn by 0x8001C824.
10. `NativeFrameGpuUpdate(0)` 0x80016634, called at 0x80011EFC.

Buffers: displayed 0x80061A7C, drawing 0x80061A80, building 0x80061A84. `Gfx_QueueObject` is 0x8001B7A0.

## `NativeFrameGpuUpdate` (0x80016634)

Advances the frame counter 0x80069034 and, while the pause word 0x8006087C is 0, steps the fade level 0x800690A0
by 0x800690A4. The sound update 0x80034310 is called at 0x800169F4 (the block from 0x800169EC is where the device
work starts); then `DrawSync` 0x8004C600, the field waits, `DrawOTag` 0x8004CBF4, `Gfx_ResetFrame` 0x8001660C, and the
frame flags reset from 0x80068F08. The fade itself is drawn by 0x800176A0 with the grey table at 0x8005C000.

## Object records

Pool pointer 0x80068E9C, 0x60 records of 0x23C bytes; the player's record pointer 0x80060A90. A record's word 0 is 1
while it holds an object; +0x44 is its list (0x80068EE0 is the free list), +0x48 its next sibling. Position +0x60,
angles +0x6C, scale +0x78, animation +0xE8 and frame +0xEC. The camera is at 0x80065CF0, the screen shake at
0x80068F00.

## `Sys_LoadCheckpoint` (0x80030388)

Kills the current zone's objects, restores the entity flags (0x300 words to 0x8006B688), the frame counter (0) and the
zone, camera path and progress from a continue block, the camera placed by `Cam_MoveAlongPath` 0x800215E8 (Crash 2's 0x800200A8). Its callers: `NativeFrameCoreLoop` at 0x80011A08
after a -2 request, 0x800307F4 (at 0x80030898) and the GOOL op 0x1C handler 0x80035D08 (at 0x80036680), the last
two without a scene request.

## Animation

- The pose fetch 0x8001B0BC(object, object + 0x78) is Crash 2's `func_8001A4A4`: when the animation +0xE8 is a vertex
  animation (first byte 1) it calls 0x8001B524(object, block + 0x20, block + 0x24) for the frame's two vertex frames
  (block = object +8) and opens the model entry named at the first frame + 0xC. Only logic paths call it (0x8001E7E4,
  0x8001F0F0, 0x800369EC).
- `Gfx_QueueObject` 0x8001B7A0, vertex kind 1: the packet takes the block's frames at +0x30 and +0x34 and its object at
  +0x10. One frame twice takes the light mode's single step (table 0x8005C270, even entries) and packet flags
  0x40000000; two frames take the morph step (odd entries: 0x8003D36C, 0x8003D748, 0x8003D788, 0x8003D80C,
  0x8003D8F8, 0x8003E39C, 0x8003F2FC) and 0x60000000.
- 0x8003F6D4 puts the halfway of the two frames' boxes (x, y halves at +0, z at +4) at scratchpad 0x118..0x11C;
  0x8003F63C copies the vertex cache and opens both frames' streams at frame + `*(frame + 0x18)` (Crash 2's 0x80043658
  with the packet's words 8 bytes on and the data offset at +0x18).
- The unlit morph reader 0x8003E0FC is Crash 2's `func_8003ED14` instruction for instruction, apart from the sum:
  x and y are `((a & lanes) + (b & lanes)) << 4` plus the box, z is `((a & 0xFF00) + (b & 0xFF00)) >> 4` plus the box
  (Crash 2: `<< 1` and `>> 7`). The draw steps load it into `$s7` (0x8003D6E0, 0x8003D8F4, 0x8003F32C, 0x8003F350);
  0x8003D0FC is the single-frame reader, and 0x800400FC a lit reader of the same shape.

## Camera modes

`Cam_Update` (0x80028A78) clears the go byte 0x8005C63F, processes the camera's messages (0x800256D4, 0x80027E68),
then runs the handler of the mode word 0x8005C634. Its values are Crash 2's `g_CameraMode` as bits (`1 << (mode - 1)`),
and each handler has its Crash 2 counterpart's shape (crash2-decomp `docs/notes/camera-cuts.md`):

| mode | Crash 2 | handler | camera |
|---|---|---|---|
| 0x10 | 5 | the kind handlers of 0x80065D80 | follows the current path |
| 0x20 | 6 | 0x80024AD4 | eases to the path at 0x8005C76C over the count 0x8005C6A8 from the position saved at 0x8005C6B0 |
| 0x40 | 7 | 0x80024E80 | follows the current path until the go byte is set; then `Cam_MoveAlongPath` 0x800215E8 puts it on the prepared path of another zone (0x8005C76C..0x8005C774) and the mode returns to 0x10: a cut |
| 0x100 | 9 | 0x80024F54 | walks the zones' paths (0x800284E8) until one is found, puts it there, mode 0x10, all in one call: a cut |
