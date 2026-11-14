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
| 0x8004411C | 0x80043FFC | core loop 0x80011CE4, scene flag 0x1000000 | a scene effect after the world |
| 0x800453F0 | 0x800452A0 | core loop 0x80011D18, scene flag 0x1000000, not levels 0xE or 0x1C | a scene effect after the world |
| 0x8003EDB8, 0x8003EEB0 | 0x8003E000 (0x8003E520 is the glow scenes' scene + 0x2C routine) | 0x8003DF3C..0x8003DFE8 in 0x8003D9DC (16 callers) | objects |
| 0x8003FE14, 0x8003FF00 | 0x8003F944 (0x8003FA00 is the underwater scenes' scene + 0x2C routine) | 0x8003F7B8 | objects |
| 0x8003DD80 | 0x8003D078 | 0x8003EBE4 in 0x8003E000 | objects |
| 0x8003CD38 | 0x8003CB24 | the pointer at 0x8005C26C (not traced) | not read |

The object rows: 0x8001B7A0 fills an object's draw packet with the routine 0x8003DEF4 or 0x8003E034 (inside
0x8003D9DC and 0x8003E000), chosen by bit 0x40000 of the object's word 0xAC.

The world renderer is host code in the psx-crash port (`game/render/world_*`, `titles/crash3/render/world_*`),
its column test against the presented canvas; the scene effects and object routines still cull at x = 512.
