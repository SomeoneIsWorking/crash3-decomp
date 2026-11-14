# World renderer — Crash Bandicoot: Warped (`SCUS_942.44`)

`Gfx_DrawWorlds` (0x80040ED8), called once a frame from the core loop (0x80011C04) with the sort list, the frame, the
scene (`D_800676B0`) and the clock, and every step it jumps through. Read from the retail disassembly. It is
Crash 2's `Gfx_DrawWorlds` (0x80041E5C, crash2-decomp `docs/notes/world-renderer.md`) with Crash 3's numbers; the
differences are listed here, the shared shape is described there. `screen-culls.md` has the screen test.

## Dispatcher

- Prologue as Crash 2 (same 0x44-byte frame, `$sp` at scratchpad 0, TRX..TRZ zeroed, clock to 0x1C, list's last and
  first entries), and ZSF3 = 0x555, ZSF4 = 0x400. An empty list clears the ordering table at frame + 0x20
  (`func_8003AC44(frame + 0x20, 0x800)`) and returns.
- Scene words: setup at scene + 0x14 (Crash 2: + 0x10), projectors + 0x18 and + 0x1C to scratchpad 0xA8 and 0xAC,
  drawers + 0x20 and + 0x24 to 0x64 and 0x68, the link tail + 0x28 to 0x70, the texture level shift + 0x30 to 0x22C.
  The setup is called with `$v1` = 0x1F800000 and `$a0`..`$a3` = setup, camera, scene, clock.
- The depth-shift table is 8 words at 0x8005D350; 0x1D8, 0x1DC and the bound 0x00D90200 at 0x21C are stored as in
  Crash 2, but the loop keeps the vertex mask in `$s6` and reloads the pair mask from 0x1DC for every vertex.
- Each entry's world bits (`entry & 0xE000`) are stored to scratchpad 0x74 as it is decoded; the bucket link tail
  reads them, so a polygon is linked by the world bits of the entry after it.
- A vertex's VXY is `(((word & 0xFFF0FFF0) >> 2) + world_xy) & 0xFFFEFFFF` and its VZ `((half & 0xFFF0) >> 2) +
  world_z`: world vertices are in quarter units.
- The colour of each corner is the word at the world's colour table plus
  `((half << 2) & 0x3C) | ((word >> 6) & 0xC00) | ((word << 6) & 0x3C0)`, as in Crash 2; Crash 3 computes the addresses
  while it loads the vertices and reads them after RTPT.
- The drain of the first entry (scratchpad 0x6C) is Crash 2's. After the loop, only the sorted link tail chains the
  slots below the lowest linked down to slot 0; for the other tails the core loop cleared the table before the call
  (0x80011BD0).

## Projectors and drawers

The default ones (`func_80011F68` stores them): triangle 0x80041648, quad 0x800418CC (0x800418C4 keeps VXY0/VZ0 at
0x1B4/0x1B8 first), triangle drawer 0x80041768, quad drawer 0x80041A3C. They are Crash 2's except:

- the depth is AVSZ3 (AVSZ4 for a quad) run before the screen test, and the slot is `0x770 − (MAC0 >> 15)` clamped at
  0 (Crash 2 sums SZ1..SZ3 and shifts by 5);
- a texture's depth level is `depth >> scratchpad[0x22C]` (Crash 2: `>> 6`);
- the drawers end through the scene's link tail (`jr` to scratchpad 0x70) with the byte count in `$a0` and the tag's
  length in `$t9`.

## Link tails

The scene picks one at scene + 0x28 (`func_80021E94`):

- 0x80041550, sorted: Crash 2's link tail (relink down to a lower slot, append at the lowest). The default.
- 0x800415A4, bucket: a polygon whose following entry is in world 0 goes to the head of slot 0 (the table's first
  word); others go to the head of the lower of their slot and the lowest so far, which becomes the lowest. No slots are
  relinked. Taken for flag 0x8000000 and for flag 0x400000 in level 0x28.
- 0x80041618, own slot: the head of its own slot. Taken when the scene flags (scene + 0x378) are negative.

## Modes

`func_80021E94` sets the scene's mode by flag (scene + 0x378), each later test overriding the earlier:

| flag | setup | projectors | drawers | what |
|---|---|---|---|---|
| — | 0x8003F518 | 0x80041648, 0x800418CC | 0x80041768, 0x80041A3C | default |
| 0x800004 | 0x80041E88 | 0x80041648, 0x800418C4 | 0x80042EF8, 0x80042F70 | glow |
| 8 | 0x80041E1C | 0x80041C64, 0x80041D28 | 0x80041CF0, 0x80041DD8 | sway |
| 0x8000000 | 0x80043004 | 0x800428D0, 0x800429E4 | 0x80042994, 0x80042ADC | underwater |
| 0x40 | 0x80041E58 | 0x80041648, 0x800418CC | 0x80042648, 0x80042764 | pulse |

Glow and underwater also store a routine at scene + 0x2C (0x8003E520 and 0x8003FA00, inside the object drawers
`func_8003E000` and `func_8003F944`); the world renderer does not read it.

- sway: Crash 2's sway. The helper 0x80041B7C is Crash 2's 0x800429D0 instruction for instruction with the angle
  `(z + phase + xy) >> 2` (Crash 2: `>> 4`) and the sine table at 0x8005C91C; the triangle's third vertex is read back
  from the GTE (Crash 2 uses the dispatcher's pair). Same instruction counts on every path.
- pulse: Crash 2's pulse with the bits swapped (bit 18 pulses, the first vertex's bit 19 dims) and the dimming in
  quarter units: black nearer than 0x96, toward black over 0xB6 (Crash 2: 0x258, 0x2D8). Same counts.
- glow: Crash 2's glow. The first corner's reach is 0x834 and the later corners' 0x514 (Crash 2: 0x15E0, 0x12C0; both
  are the first less 0x320); the light is shifted left by 2 before the tint is added (one more instruction), the
  corner's colour is kept at scratchpad 0xDC (Crash 2: 0xC8), and the black flat polygon's tail loads the link tail
  (one more instruction each). The later corners go through a routine pointer in `$t2`: 0x80042BA4 both lamps,
  0x80042B4C one lamp (moved into `$t3`/`$t0`), 0x80042B44 none.
- underwater (Crash 3 only): each corner with bit 18, or with neither bit 18 nor 19, runs the sway helper; bit 18's
  vertices move, and the helper's last sine halfword (`$t9`, the one that moved the pair) is kept at scratchpad
  0x190 + 4 × corner. The triangle's third vertex and the quad's fourth come from the dispatcher's pair. The drawers
  load IR0 from that word and RGBC from the colour for every corner and DPCS the corners without bit 19 toward the far
  colour the setup took from scene + 0x360. The last corner's result is read by the `mfc2` in the delay slot of the branch back to the
  default drawer (0x800429E0, 0x80042B40).

## Reached by warp

At each level's entry (`warp scene`, 6 s): default in most scenes; sway 0x0B, 0x0C, 0x10; underwater (bucket tail)
0x0E, 0x1C; glow 0x24, 0x25. Pulse and the own-slot tail are selected at no level entry. Scene 0x3A aborts the
process on its warp.
