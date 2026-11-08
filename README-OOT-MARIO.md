# Lichii: Ocarina of Time build (mariohbd)

A personal build of [Lichii](https://github.com/zortos293/Lichii) (GPL-3.0) tuned for
*The Legend of Zelda: Ocarina of Time* on Switch 2 (`0400486024DEC000`), tested on a
Ryzen 5 5600X, RTX 3060 Ti 8 GB and a 3440x1440 monitor.

It contains no game files, keys or firmware. You need your own legally dumped copy of the game and
your own keys.

## Where the code comes from

All of the work here is other people's. This branch puts it together and adds small switches to
turn each part off.

| Part | Source |
| --- | --- |
| Base | Lichii `nx2` with [#717](https://github.com/zortos293/Lichii/pull/717): split GPU thread, caches, JIT and kernel work |
| Image recycler, memory priorities, held-draw pipeline builds, present and audio thread priorities | Lichii PRs and the #717 follow-ups |
| Host zstd for the game's files | [#869](https://github.com/zortos293/Lichii/pull/869) |
| Unbacked image fix (`EDEN_NX2_IMAGE_MAY_FAIL`) | [#880](https://github.com/zortos293/Lichii/pull/880) |
| NUP zstd members decoded only as far as reads need | [#885](https://github.com/zortos293/Lichii/pull/885) |
| FMZ before SAT, the fix for the white lights in Gerudo Valley | [#811](https://github.com/zortos293/Lichii/pull/811) |
| Movie sync, the dma_pusher race, mip chains, pipeline cache versions, vi2/nifm stubs, the noperspective fix | [#571](https://github.com/zortos293/Lichii/pull/571), [#727](https://github.com/zortos293/Lichii/pull/727), [#820](https://github.com/zortos293/Lichii/pull/820) and others listed in the commits |
| FPS Unlock built-in and the frame rate limit slider | [#655](https://github.com/zortos293/Lichii/pull/655) (off by default here, see below) |
| Vertex attribute mask for pipeline keys (`EDEN_NX2_ATTR_MASK`) | [#408](https://github.com/zortos293/Lichii/pull/408) |
| Water flicker fix (`EDEN_NX2_WATER_FIX`) | [#778](https://github.com/zortos293/Lichii/pull/778), from Frizti's [Fritchii](https://github.com/Frizti/Fritchii) |
| HUD at the screen edges with the 21:9 mod (`EDEN_OOT_UW_HUD_EDGES`) | [#831](https://github.com/zortos293/Lichii/pull/831) (commit `e0053016c2`) |
| Native uniform bank upload skipped (`EDEN_NX2_OOT_BANK_SKIP`) | [#909](https://github.com/zortos293/Lichii/pull/909) |
| Cached pipelines prebuilt newest first | [#875](https://github.com/zortos293/Lichii/pull/875), from Fritchii |
| Submit while the front waits, pending wake | A-Man-With-A-Plan-dev/Eglantine |
| Skip texture alias sync when no alias changed | 1Shad0w/Lichii |

Each commit names its source PR or fork commit. The full list is in this branch's history on top of
Lichii `nx2`.

Ultrawide is **not** built in. It comes from Ivan0xFF's mod,
[craftycorvid/oot-mod](https://github.com/craftycorvid/oot-mod) (MIT). With that mod enabled, this
build also moves the hearts and the item buttons to the edges of a 3440x1440 screen.

## Changes from upstream defaults

- **FPS Unlock (built-in)** is off. "Dynamic FPS by Dark" applies instead. In our testing, FPS Unlock
  was the likely cause of a game panic about 2.5 minutes in. `EDEN_NX2_OOT_FPS_UNLOCK=1` turns it on.
- **Occlusion Off (built-in)** is off. It cost frame rate and brought white lights in Gerudo Valley.
  `EDEN_NX2_OOT_OCCLUSION_OFF=1` turns it on.

## Switches

Each new part can be turned off with a line in `nx2_switches.txt` beside `lichii.exe`, followed by
a restart:

```
EDEN_NX2_WATER_FIX=0
EDEN_NX2_ATTR_MASK=0
EDEN_NX2_IMAGE_MAY_FAIL=0
EDEN_NX2_OOT_BANK_SKIP=0
EDEN_OOT_UW_HUD_EDGES=0
```

## Building

Use the same steps as Lichii on Windows: MSVC 2022, the Vulkan SDK and Qt 6.8. The release has two
builds:

- `lichii-windows-x86_64-msvc`: MSVC Release, for any x86-64 CPU.
- `lichii-windows-x86_64-znver3-pgo`: clang-cl with ThinLTO, PGO (Lichii's profile) and
  `-march=znver3`. It needs a Ryzen 5000 or newer, or another CPU with the same instruction set.

## License

GPL-3.0-or-later, the same as Lichii, Eden and yuzu. See `LICENSE.txt`.
