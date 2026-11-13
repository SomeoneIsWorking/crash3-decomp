# Boot and measured facts — Crash Bandicoot: Warped (`SCUS_942.44`)

Moved from psx-crash `titles/crash3/README.md`. `executable.json` and `entry/facts.cpp` are psx-crash paths.

This title targets North American `SCUS-94244`, executable `SCUS_942.44`. The retail disc's
`SYSTEM.CNF` selects that file; the unrelated `DRAGON/SPYRO.EXE` on the same disc is never accepted as
the product image. psx-crash `titles/crash3/executable.json` owns the measured identity and PS-X EXE facts.

Recorded execution and the independent CPU agree 34/34 at the first eight calls through game main
`0x80048AA0` and `EnterCriticalSection` wrapper `0x80048C38`. Cause `0x20`, EPC `0x80048C3C`,
resume `0x80048C40`, and the following B0 dispatch agree. Structural target `0x800112B8` is not an
A(39h) thunk; that modeled return is refused.
