# Crash Bandicoot: Warped decomp

A decompilation of **Crash Bandicoot: Warped** (USA, `SCUS-94244`, executable `SCUS_942.44`), set up with
[splat](https://github.com/ethteck/splat). It currently holds the build-matched split configuration of the boot
executable, a symbol map from our own reverse engineering, and the RE notes. No C is matched yet.

The PC port [psx-crash](https://github.com/SomeoneIsWorking/psx-crash) consumes this repository as a submodule and
reads its guest addresses from the symbol map.

## Bring your own disc

No game bytes are ever committed here: not the executable, not the split assembly or data. Copy `SCUS_942.44` from
your own disc into `disc/` (gitignored):

```
disc/SCUS_942.44
```

The config carries the file's SHA-1 (`ffa9252384eb16ce5ae14061bb87c86701ab4893`) and splat refuses a mismatch. The
disc's `DRAGON/SPYRO.EXE` is a different program and is not split here.

## Split

```
uv run --frozen splat split config/scus_942.44.yaml
```

Output goes to `asm/` and `build/`, both gitignored. splat is pinned in `pyproject.toml`/`uv.lock`.

## Layout

| path | what |
|---|---|
| `config/scus_942.44.yaml` | the splat config |
| `config/symbols/scus_942.44.txt` | the symbol map, splat `symbol_addrs` format |
| `docs/re-frontier.md` | the ordered RE evidence chain (`re_frontier.py`) |
| `docs/notes/` | behaviour notes on guest routines |
| `docs/our-names.md` | where each name comes from, and the code sites the port uses |
| `docs/prior-art.md` | other Crash 3 work, licences and what we take |

Notes cite psx-crash paths (`game/...`, `titles/...`, `docs/issues/...`) where a port owner or an issue holds the
evidence.

## Licence

MIT (`LICENSE`), covering this repository's own files only. Crash Bandicoot: Warped is © Sony Interactive
Entertainment / Naughty Dog; nothing of it is distributed here.

Built with AI assistance (Claude and other LLMs).
