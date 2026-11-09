# Lichii OoT + shader precache (#717 + #579)

A Windows build of [Lichii](https://github.com/zortos293/Lichii) for **The Legend of Zelda: Ocarina
of Time** on Switch 2 (`0400486024DEC000`). It compiles the game's shaders **before** you play, so new
areas no longer freeze while their shaders build.

It starts from mariohbd's [Ocarina of Time build v1](https://github.com/mariohbd/Lichii/releases/tag/oot-mario-v1)
(Lichii `nx2` + [#717](https://github.com/zortos293/Lichii/pull/717), the split GPU thread, cache and JIT
work that helps CPU-bound PCs) and adds [#579](https://github.com/zortos293/Lichii/pull/579): graphics
pipeline libraries and a shader warm-up built from your own game files.

No game files, keys or firmware are included. You need your own legally dumped game and keys.

**Download:** [Releases](https://github.com/ibsorn/Lichii/releases), `lichii-oot-579-windows-x86_64-msvc.zip`.

## What changes in play

On the test PC (below), same game and settings, first visits to areas:

| | Before (OoT build v1, no libraries) | This build, after the warm-up |
|---|---|---|
| Worst 10 s of shader building on the GPU thread | 1,912 ms | 173 ms |
| Worst 10 s of draws waiting for unbuilt shaders | 2,658 ms | 2 ms |
| Longest single shader build | 229 ms | 77 ms |

From the emulator log (`ReportRuntimePipeline`). Not a controlled A/B: the "before" is about 5 minutes
of play, the "after" about 7 minutes in partly different areas. The other #579 testers report the same
trend (multi-second freezes on first visits gone).

## The first launch takes a while (once)

With *Precache shaders before game start* on, the first launch builds every shader it can find in the
game files on the loading screen:

| Step | On the test PC (8 threads) |
|---|---|
| Read the game files and write the warm-up files | 10 s (4,794 compute, 22,594 vertex, 37,402 fragment programs) |
| Compute shaders | 4,660 built, 0 failed, 4 min |
| Graphics shader libraries | 59,324 built, 672 failed (0 by the driver), 31 min |

The 672 failures are programs whose state can't be guessed from the files; other #579 testers get the
same 59,324 / 672 split. They are built during play as usual.

Later launches find the work done ("already done for this driver" in the log) and start in about a
minute. **The warm-up runs again (once) after a GPU driver update**, a different GPU, or a deleted
shader cache. It needs about 1.7 GB in `%APPDATA%\eden\cache\shader\0400486024dec000` plus about 1 GB
in the NVIDIA driver cache.

## Setup

1. Unzip anywhere and run `lichii.exe`. Settings, keys, saves and caches are shared with Lichii, Eden
   and other Lichii forks (`%APPDATA%\eden`).
2. Right-click the game > **Properties** (per-game settings), and set:

| Where | Setting | Value |
|---|---|---|
| Graphics > Advanced | **Shader libraries (faster first-time shader builds)** | On |
| Graphics > Advanced | **Precache shaders before game start** | Always |
| Graphics > Advanced | Switch 2 draws with unbuilt pipelines | Bounded wait (8 ms) |
| Graphics > Advanced | GPU accuracy | Fast |
| Graphics | Resolution | 1x |
| System | Console mode | Docked |
| Add-Ons | Dynamic FPS by Dark (built-in) | On |
| Add-Ons | FPS Unlock (built-in) | Off |
| Graphics | Use NVIDIA DLSS in place of a game's DLSS | Optional (RTX). Needs the Streamline files from the button next to it |

3. Start the game and leave the loading screen alone until it finishes (don't press *Skip*).

If you open the game with another Lichii build, that build drops the *Shader libraries* setting it
doesn't know; turn it on again here.

## Will it work on my PC?

Honest answer: **it has been tested on one PC**. What the code does elsewhere is known; how well it runs
is not.

| | Status |
|---|---|
| **Tested** | Intel i7-6700K (4 cores / 8 threads), NVIDIA RTX 3070 8 GB, 32 GB RAM, Windows 11, NVIDIA driver 610.88 |
| **CPU** | Any x86-64 CPU (plain MSVC build: no AVX-512, no Zen-only code). Ocarina is CPU-bound, so frame rate depends mostly on the CPU (not measured on the test PC); fewer threads also make the warm-up longer |
| **NVIDIA RTX 20/30/40/50** | Expected to work like the test PC (same driver path) |
| **NVIDIA GTX 10 (Pascal)** | Untested. Mesh scenes go through compute emulation and keep the old (no-library) path |
| **AMD Radeon** | Untested here. #579 links libraries optimized on AMD's Windows driver (fix for white fringes). An open report on #717 has RDNA2 (RX 6000) losing the device at save load; RX 6000 and older also skip the matrix-unit path |
| **Intel Arc / iGPU** | Untested. Libraries turn on only when the driver offers `VK_EXT_graphics_pipeline_library` with fast linking; otherwise they stay off (the log has no `graphics pipeline libraries on` line) and the precache builds only the compute shaders and the already cached pipelines |
| **RAM** | The emulator used about 8 GB during the warm-up on the test PC; 16 GB or more is advisable |
| **Linux, macOS, Android** | No builds from this branch |

If it doesn't run well on your hardware, the log (`%APPDATA%\eden\log\eden_log.txt`) shows what
happened: look for `Nx2Perf gpl`, `Shader precache` and `Runtime pipelines in the last 10 s`.

## Switches

`nx2_switches.txt` beside `lichii.exe` keeps mariohbd's tuning (split GPU thread, preflush, JIT
superblocks, host zstd, Occlusion Off disabled) and documents #579's: `EDEN_NX2_WARM_THREADS`
(warm-up threads), `EDEN_NX2_COMPUTE_NOINLINE` / `EDEN_NX2_GRAPHICS_NOINLINE`. `EDEN_NX2_GPL=1` turns
the libraries on without the setting.

## How it was put together

- Base: mariohbd/Lichii `claude/oot-build` at `eb74c32b2`, the tree of OoT build v1.
- #579's own 26 commits (not the `nx2` merges it carries) applied three-way. Ten conflict blocks in six
  files were resolved by hand, keeping #717's code paths and adding #579's; see the commit message of
  `4c3372877` for each one.
- One follow-up: this branch's `FileEnvironment::Deserialize` takes no end bound, so the graphics
  precache checks the file position itself.
- Built by [`.github/workflows/oot-579-windows.yml`](.github/workflows/oot-579-windows.yml): Windows,
  MSVC Release, no ThinLTO (a ThinLTO build of #717 is reported to hang at boot), update checker off so
  it doesn't replace itself with an upstream Lichii build.

## Credits

All of the emulation work is other people's: the [Lichii](https://github.com/zortos293/Lichii), Eden
and yuzu developers; mariohbd for the OoT build; stephenjayakar for #717; PantACRO4life for #579, with
daniel-lynch, justin-wq1337, jarynclouatre and the other contributors named in its commits.
Dynamic FPS by Dark.

mariohbd's README for OoT build v1 is in [README-OOT-MARIO.md](README-OOT-MARIO.md), Lichii's in
[README-LICHII.md](README-LICHII.md).

## License

GPL-3.0-or-later, like Lichii, Eden and yuzu. See [LICENSE.txt](LICENSE.txt).
