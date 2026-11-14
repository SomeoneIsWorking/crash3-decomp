# Water grid — Crash Bandicoot: Warped (`SCUS_942.44`)

0x80043FFC and the routines it jumps through (0x8004420C, 0x8004447C, 0x80044674, 0x80044878), hand-written, about
830 instructions. Read from the retail disassembly. The core loop calls it after the world (0x80011CE4) when the scene
flags (scene + 0x378) have 0x1000000, with `$a0` the frame clock (`D_80069034`), `$a1` the frame and `$a2` the scene.
It draws the sea of the jet-ski levels as a grid of textured, semi-transparent quads around a centre point.

## Setup

- Saves `$s0`..`$s7`, `$t8`, `$t9`, `$gp`, `$fp` in a 0x40-byte frame; `$sp` goes to scratchpad 0 and `$ra` to 0x384
  (`$sp` itself is used as a variable below). ZSF3 = 0x555, ZSF4 = 0x400, TRX..TRZ = 0.
- The phase is `clock % 0xE0`: its low 3 bits are the blend between two texture frames, its `>> 3` picks the frame
  pair from the table at scene + 0xB8 (frame = word at scene + 0xB8 + 4 × (phase >> 3), next frame the word after).
  The blend, the frame and the next frame are stored at scene + 0xA0, 0xA4, 0xA8; the blend << 9 at scratchpad 0x398.
- The texture record at scene + 0xB4: its word 4 (CLUT and base coordinate) to scratchpad 0x370 and its low half to
  0x37C; word 8 (page) with the semi-transparency bits set to mode 1 (`& 0xFF9FFFFF | 0x200000`) to 0x374; word 12 to
  0x378.
- The centre is the object at scene + 0x0C, its position at + 0x14, 0x18, 0x1C. With x' = x − 0x6400 and
  z' = z − 0x6000: the map offsets are x' >> 10 and z' >> 10 (logical), the grid's origin is
  `(−0x6000 − (x' & 0x3FF)) >> 2` on each axis (arithmetic), so the grid steps in 0x100 units and keeps the centre
  in its middle cell. The water level is `(scene[0x9C] − y) >> 3`. The height map is scene + 0xAC (0 for none).
- The packed screen bound 0x00D90200 goes to scratchpad 0x38C, the per-column valid flags (below) are cleared, and the
  camera's heading (scene + 0 → + 0x10) plus 0x200 picks the quadrant: bit 11 the far half of the grid, bit 10 across
  the rows. Across the rows the map x offset is shifted left by 8, otherwise the z offset is.

| bit 11 | bit 10 | rows | vertex routine | cell drawer |
|---|---|---|---|---|
| 0 | 0 | 0..0x18, columns ascending | 0x800446B0 | 0x80044878 |
| 0 | 1 | 0..0x18, columns ascending | 0x80044674 | 0x80044908 |
| 1 | 0 | 0x30..0x18, columns descending | 0x800446B0 | 0x800448BC |
| 1 | 1 | 0x30..0x18, columns descending | 0x80044674 | 0x80044960 |

The vertex routine's address is kept at scratchpad 0x380; the cell drawer stays in `$t4`.

## Rows

The near half calls 0x8004420C for rows 0..0x18 (`$s1` = row << 8), each row's first and last column the byte pair at
0x8005D31C + 2 × row; the far half calls 0x8004447C for rows 0x30 down to 0x18, reading the pair at
0x8005D37E − 2 × row swapped (the near table's pair for row 0x31 − row, so row 0x18 reads the terminating (0, 0)).
The pairs trace a circle of radius 24 cells: (19, 29), (16, 32), ... (0, 48) at row 24. The two row routines are mirror
images; the near one:

- The previous row leaves in `$sp` the column of its last cell culled before its first drawn one (`$sp` starts 0 near,
  0x30 far). A row whose first column is left of it starts there instead, and one column further back when that
  column is at least 0x17 (far: at most 0x19), clearing that column's valid flag.
- The first column's points at the row and the next row (row + 0x100 near, − 0x100 far) are loaded through the vertex
  routine. Then for each further column toward the last: the point at the row and at the next row. With the two of the
  column before, they make the cell A (column before, row), B (column before, next row), C (row), D (next row): A's
  screen word is stored to scratchpad 0x390 from SXY0 between the two calls, B, C and D are SXY0..SXY2 after them.
- The cell is culled by the packed test (`screen-culls.md`) on A..D, with AVSZ4 run in its middle. A cell before the
  first drawn one that is culled sets `$sp` to its column and the walk goes on; a culled cell after a drawn one ends
  the row. A visible cell calls the drawer with `$a1` = 0x770.
- After each cell C and D become the next cell's A and B: their colours (`$s4` and scratchpad 0x394 into `$s2`,
  `$s3`) and texture coordinates (halves 0x378, 0x37A into 0x370, 0x374).

## Vertex routine

0x80044674 (across the rows) and 0x800446B0 (along them) take the column in `$t8`, the row in `$t9` and a mask in `$a1`
(−1 for the row's own point, 0 for the next row's). They keep a cache per column in the scratchpad: the texture
coordinate at 0x124 + 4c with the valid flag in the halfword above it, the colour at 0x60 + 4c, the screen word at
0x1E8 + 4c and the depth at 0x2AC + 4c (columns 0..48). The next row's point is always projected and cached; the row's
own point is taken from the cache when its valid flag is set, so each point is projected once.

- Cached: SXY1, SZ2 to SXY0, SZ1; SXY2, SZ3 to SXY1, SZ2; the cached screen and depth into SXY2, SZ3 (SZ0 is left as
  it was, unlike RTPS). Returns the cached colour in `$t8` and coordinate in `$t9`.
- Projected: along the rows the point is (column << 8 + origin x, row + origin z) and its map cell (column + map x,
  row + map z); across them (row + origin x, column << 8 + origin z) and ((row + map x) >> 8, (column + map z) << 8).
  The map byte is read at `(x & 0xFF) | (z & 0xFF00)`, and from row 0 when z's row is 0xFF; with no map it is 0x10.
- The cell's word in each of the two texture frames' tables (32 × 32 words, index `(x & 0x1F) | ((z & 0x1F00) >> 3)`)
  holds the height in its top byte and a texture coordinate as its colour bytes. The frame's word goes to RGBC, the next
  frame's bytes to the far colour (each `<< 4`), the blend to IR0, and DPCS blends the two; the blended RGB2 plus the
  base coordinate is the point's texture coordinate. The height is
  `(map & 0x1F) × ((h0 << 3) + ((blend × (h1 − h0)) >> 9)) >> 7` plus the water level, put in VXY0's y as `<< 17`.
- After RTPS the colour is a grey from RGB2's third byte: with d = max(|x|, |z|) + min(|x|, |z|) / 4 − 0x800, the byte
  >> 1 when d < 0, `(0x1000 − d) × byte × 16 >> 17` while d < 0x1000, black beyond. The screen word, SZ3, colour and
  coordinate are cached and the flag set.

## Cell drawer

The four entries read the cell's map byte at the cell's own corner for the facing (along near: (column + map x,
row + map z); along far: one column on and one row back; across near: one row on and one column back in map terms;
across far: the corner itself), then:

- Bit 7 set: half the cell as a GT3 (command 0x36). Bits 6 and 5 of the byte, turned by the facing (along far adds
  0x40; across near takes 0x80 − (byte & 0xE0); across far 0xC0 − (byte & 0xE0)), pick BCD (00), ABD (01), ABC (10) or
  ACD (11). A triangle whose three colours are all black is skipped, except ACD, whose test ORs the shifted bits in
  place of A's colour and so never skips.
- Otherwise bits 7..5 zero skip the cell; nonzero, or no height map at all, draw the whole cell as a GT4 (command
  0x3E), skipped when all four colours are black.
- The slot is `0x770 − (MAC0 >> 15)` clamped at 0, times 4 and clamped at 0x1FFC, in the frame's ordering table at
  frame + 0x20; the packet goes to the slot's head and `$s7` moves past it (0x34 a quad, 0x28 a triangle). The
  texture words are scratchpad 0x370 (CLUT, A), 0x374 (page, B), 0x378 (C, D), the CLUT and page halves rejoined with
  B's or C's coordinate for the triangles without A.

## Screen test and the margins

Only the cell test uses the bound at scratchpad 0x38C, as the world's projectors do: the left edge is the SXY sign bit,
so a bound cannot widen it. Because the walk's start and end depend on which cells are culled, a wider test changes
the cells visited, the column cache and the GTE as well as the packets.
