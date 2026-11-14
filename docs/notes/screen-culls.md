# Screen-edge culls — Crash Bandicoot: Warped (`SCUS_942.44`)

Read from the retail executable's disassembly.

The packed screen bound 0x00D90200 (217 << 16 | 512, Crash 2's world bound) is built inline by `lui`/`ori` pairs at
ten sites: 0x8003CD38, 0x8003DD80, 0x8003EDB8, 0x8003EEB0, 0x8003FE14, 0x8003FF00, 0x8004103C, 0x800436AC, 0x8004411C
and 0x800453F0.

At 0x8003CD38 the routine projects three corners and rejects the polygon (branch to 0x8003CDB8, its loop's next
entry) when every corner is past the same edge, as Crash 2's world projectors do: `nor` of the three SXY words less
the bound sets a half's sign when each corner is at or beyond the right (x) or bottom (y) bound, the AND of the three
words sets it when each is left of or above 0, and bit 31 then bit 15 are tested. The left and top edges are the
sign bits of the SXY words, so a constant cannot move them. Every site tests its corners the same way.

Where each site is (addresses as SCUS_942.44 places them):

| site | routine | reached from | draws |
|---|---|---|---|
| 0x8004103C | `Gfx_DrawWorlds` 0x80040ED8 (`world-renderer.md`) | core loop 0x80011C04 | the world's sort list |
| 0x800436AC | 0x80043310 | core loop 0x80011CC0, scene flag 0x400000 and scene + 0x98 set | a scene effect after the world |
| 0x8004411C | 0x80043FFC | core loop 0x80011CE4, scene flag 0x1000000 | the water grid (`water-grid.md`) |
| 0x800453F0 | 0x800452A0 | core loop 0x80011D18, scene flag 0x1000000, not levels 0xE or 0x1C | a scene effect after the world |
| 0x8003EDB8, 0x8003EEB0 | the model drawer from 0x8003E040 (0x8003E520 is the glow scenes' scene + 0x2C routine) | object draw paths | objects |
| 0x8003FE14, 0x8003FF00 | 0x8003F944 (0x8003FA00 is the underwater scenes' scene + 0x2C routine) | 0x8003F7B8 | objects |
| 0x8003DD80 | 0x8003D078 | 0x8003EBE4 in 0x8003E000 | objects |
| 0x8003CD38 | 0x8003CB24 | the pointer at 0x8005C26C (not traced) | not read |

The model drawers test each triangle the same way (after an NCLIP when bit 28 of its word asks for one), so a triangle with every corner past one edge is dropped.

## Object box test

0x8001B7A0 stores the packet's visibility test at packet + 0x24: `D_8003E034` (always visible, `t8` 0) when bit
0x40000 of the object's word 0xAC is set, otherwise `D_8003DEF4`. The object draw paths call it with `jalr $t8`
(0x8003D30C, 0x8003DC68, 0x8003E724, 0x8003F310, 0x8003FC64) and skip the model when `t8` comes back nonzero. It is
Crash 2's 0x80041D14 in quarter units:

- `$v1` is the scratchpad; the test keeps the caller's `$ra` at 0x64 and reads the box corner from 0x118 (packed
  x, y) and 0x11C (z). The extent is 0x1FE0 on each axis, x masked by `$t2`, `$t9` = 0x1FE00000 throughout.
- Three RTPTs, corners in this order: (x, y, z), (x+e, y, z), (x+e, y+e, z); (x, y+e, z), (x, y+e, z+e),
  (x, y, z+e) with VZ0 kept; then VXY1 and VXY2 become (x+e, y) and (x+e, y+e) and only those two are tested.
- Each corner (SXY in `$a0`, the matching SZ in `$a1`) goes through 0x8003E000 by `jal`: it returns to the next
  corner when SY < 0, SX < 0, SZ < 0xC8, SY >= 0xD8 or SX >= 0x200, and otherwise loads `$ra` from 0x64 and returns
  from the whole test with `t8` 0. After all eight, `t8` is -1. Every exit returns to the entry `$ra`.

## Seen in the picture at 16:9

The world renderer and the water grid are host code in the psx-crash port, their column test against the presented
canvas. The rest still cull at x = 512; whether that shows was tested by raising each group's right bound through the debug server (the `ori`
after each `lui` to 0x3FF; the box test's `addiu $a2, $a2, -0x200` at 0x8003E024 to -0x3FF) from one saved state,
16:9, 60 frames later, against the same state unpatched (scenes 0x02, 0x03, 0x04, 0x0D, 0x0E):

- The scene effects: the jet-ski level 0x0D fills 2778 more pixels of its right margin with the sea, which stops at a
  jagged edge without it; nothing else changes. Raised one at a time, all of it is 0x80043FFC's (its bound is the word
  it stores at scratchpad 0x38C, which its cell routines 0x8004420C, 0x8004447C, 0x80044674 and 0x80044878 read);
  0x800452A0's and 0x80043310's (scene 0x02) change nothing in the scenes tried. The left edge is the sign bit, so a
  bound cannot show it.
- The object box test and the model drawers' triangle test: no margin pixel changes in any of the five scenes. With
  both raised, the game's own timing changes (the extra guest drawing), so later frames differ everywhere: 0x04 is
  out of phase 150 frames on.

## The world's sort list

At 16:9 the margins of Dingodile (0x03) and the underwater level (0x0E) still show the clear colour in places: on the
right of Dingodile's arena, in wedges down both sides of 0x0E. Logged per frame (each projected polygon's corners
rasterised into 4-pixel cells, kept or not), no polygon in the list covers those cells; the only margin cells the
renderer drops are below row 217, the retail row test. The SLST list names only what the 4:3 view needs there.

