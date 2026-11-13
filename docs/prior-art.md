# Prior art — Crash Bandicoot: Warped (`SCUS_942.44`)

Outside work on `SCUS_942.44` (SHA-1 `ffa9252384eb16ce5ae14061bb87c86701ab4893`); a reference names addresses, our measurement decides behaviour.
Address checks ran psxport `decomp_pipeline.py --function-at` on our image (2026-10-13) unless marked "our docs";
licences, trees and push dates were re-read through the GitHub API on 2026-11-13. Paths such as `docs/issues/...` are
psx-crash paths.

Tools that cover all three titles (GOOL bytecode, NSF/NSD data) are listed in `crash-bandicoot-decomp`'s
`docs/prior-art.md`.

## Decomp

| repository | URL | licence | target build and match | progress | what we may take |
|---|---|---|---|---|---|
| `crash3-decomp` (this repository; psx-crash `external/crash3-decomp`) | https://github.com/SomeoneIsWorking/crash3-decomp | MIT | `config/scus_942.44.yaml` carries our SHA-1; splat refuses any other build | split config, symbol map from the port, RE notes; no C | everything (ours) |

## Other projects

| project | URL | kind | licence | target build | build match (addresses checked) | what we may take | notes |
|---|---|---|---|---|---|---|---|
| NyperYuhgard/Crash-Bandicoot-Warped | https://github.com/NyperYuhgard/Crash-Bandicoot-Warped | Ghidra project plus armips asm patches | none | Crash 3 `SCUS_942.44` | matched: the committed retail exe blob equals ours; `Game_Initialize_And_Boot 0x80011628` calls `0x80048AA0` and `0x800154AC` | evidence only (names from the `.s` files) | Not a decomp: no C or splat config. Distributes the retail executable and `armips.exe`; do not download or open its `.gpr`/`.rep`. Last push 2026-08-04. |
| alexbeavs-ps1-ports/crash-bandicoot-3-warped-recomp | https://github.com/alexbeavs-ps1-ports/crash-bandicoot-3-warped-recomp | psxrecomp static recompilation | PolyForm-Noncommercial-1.0.0 (`LICENSE`) | Crash 3 `SCUS-94244` | matched: `seeds/ghidra_funcs.txt` contains our `0x80048AA0`, `0x800154AC`, `0x80011628`, `0x8004F37C`, `0x8004B2A8` | nothing from the runtime; seeds as a census | Last push 2026-11-13. |
| omegakatana92/CrashWarpedRecomp | https://github.com/omegakatana92/CrashWarpedRecomp | psxrecomp config and seeds | none | Crash 3 USA (README) | not checked | nothing from the runtime | Last push 2026-09-03. |
| Purplesmaug96/c3-recomp | https://github.com/Purplesmaug96/c3-recomp | psxrecomp static recompilation | MIT | Crash 3 (no serial named) | not checked: no address list | nothing from the runtime | Last push 2026-08-02. |

Not found (2026-11-13): a community decomp of `SCUS_942.44`; a widescreen or 60 fps patch for Crash 3; any Crash 3 entry
on decomp.dev. `dguillot-gh/crash3-android` is gone (404).
