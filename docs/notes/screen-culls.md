# Screen-edge culls — Crash Bandicoot: Warped (`SCUS_942.44`)

Read from the retail executable's disassembly.

The packed screen bound 0x00D90200 (217 << 16 | 512, Crash 2's world bound) is built inline by `lui`/`ori` pairs at
ten sites: 0x8003CD38, 0x8003DD80, 0x8003EDB8, 0x8003EEB0, 0x8003FE14, 0x8003FF00, 0x8004103C, 0x800436AC, 0x8004411C
and 0x800453F0.

At 0x8003CD38 the routine projects three corners and rejects the polygon (branch to 0x8003CDB8, its loop's next
entry) when every corner is past the same edge, as Crash 2's world projectors do: `nor` of the three SXY words less
the bound sets a half's sign when each corner is at or beyond the right (x) or bottom (y) bound, the AND of the three
words sets it when each is left of or above 0, and bit 31 then bit 15 are tested. The left and top edges are the
sign bits of the SXY words, so a constant cannot move them. The other nine sites are not yet read.
