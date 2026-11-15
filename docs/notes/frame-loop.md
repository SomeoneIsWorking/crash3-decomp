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
zone, camera path and progress from a continue block (0x800215E8). Its callers: `NativeFrameCoreLoop` at 0x80011A08
after a -2 request, 0x800307F4 (at 0x80030898) and the GOOL op 0x1C handler 0x80035D08 (at 0x80036680), the last
two without a scene request.

## Open

- `Cam_Update`'s handlers (mode word 0x8005C634, kind 0x80065D80) are not mapped against Crash 2's prepared cut and
  zone walk; which of them move the camera in one step is not known.
- Vertex-animated objects' frame fetch and morph drawer are not read.
