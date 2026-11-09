# Lichii: Ocarina of Time build

An unofficial build of the [Lichii](https://github.com/zortos293/Lichii) emulator, tuned for
**The Legend of Zelda: Ocarina of Time** on Switch 2 (`0400486024DEC000`, `main` build
`E0F3CCFAF50065AE4EA01EB5AEB13B1A45DA3348`).

It was put together and tested on a Ryzen 5 5600X, RTX 3060 Ti 8 GB and a 3440x1440 monitor. It is
not affiliated with Nintendo or with the Lichii project. No game files, keys or firmware are
included: use only a copy of the game and system files you obtained legally.

**[Download the latest release](https://github.com/mariohbd/Lichii/releases/latest)**

The original Lichii README is in [README-LICHII.md](README-LICHII.md).

---

## What this build changes

### Visual fixes

| Problem | Fix | Source |
| --- | --- | --- |
| Water flickers every other frame (Lake Hylia, Zora's River) | The water shader reads a light's texture handle per pixel. The translation fell back to a buffer address the game swaps every frame, so the texture changed every frame. Such a handle now binds no texture. | [#778](https://github.com/zortos293/Lichii/pull/778), from Frizti's [Fritchii](https://github.com/Frizti/Fritchii) |
| Blown-out white lights on the walls in Gerudo Valley | FMZ is applied before SAT in FFMA and HFMA2, and half-precision FMZ is translated. | [#811](https://github.com/zortos293/Lichii/pull/811) |
| White lights and lower FPS with "Occlusion Off" | The built-in is off by default. | this build |
| Red line in the Royal Family's Tomb | Noperspective varyings are interpolated perspective-correct. | [#820](https://github.com/zortos293/Lichii/pull/820) |
| Textures left without memory when VRAM runs out | Creating such an image now fails, instead of leaving it without memory. | [#880](https://github.com/zortos293/Lichii/pull/880) |

### Stability

- **Game panic about 2.5 minutes in.** "FPS Unlock (built-in)" ([#655](https://github.com/zortos293/Lichii/pull/655))
  was the likely cause, so it is **off by default**. "Dynamic FPS by Dark" applies instead, as in
  earlier builds.
- Movie video stays in sync with its audio (imports [#571](https://github.com/zortos293/Lichii/pull/571)).
- The DMA pusher's fence thread no longer overwrites `dma_increment_once`. The vi2 and nifm commands
  the game calls are stubbed ([#727](https://github.com/zortos293/Lichii/pull/727)).
- A texture alias's newest tick stays at least as new as each alias's modification, so stale aliases
  are not skipped.

### Performance

| Change | What it does | Source |
| --- | --- | --- |
| Split GPU thread, caches, JIT, kernel work | The main Lichii performance work for this game | [#717](https://github.com/zortos293/Lichii/pull/717) |
| Host zstd | The emulator decompresses the game's `.zs` files itself, for faster loading | [#869](https://github.com/zortos293/Lichii/pull/869) |
| NUP zstd on demand | Package members are decoded only as far as reads need | [#885](https://github.com/zortos293/Lichii/pull/885) |
| Vertex attribute mask | Pipelines are no longer split by vertex attributes the shader never reads, so fewer pipelines are built and there is less stutter | [#408](https://github.com/zortos293/Lichii/pull/408) |
| No once-a-frame bank upload | Unchanged 64 KiB uniform banks are not uploaded again every frame (about +3% FPS in the author's test) | [#909](https://github.com/zortos293/Lichii/pull/909) |
| Newest pipelines first | Cached pipelines are prebuilt newest first, which helps after a driver update | [#875](https://github.com/zortos293/Lichii/pull/875), from Fritchii |
| Submit while waiting, pending wake | The GPU sits idle less | A-Man-With-A-Plan-dev/Eglantine |
| Alias sync skip | No texture alias sync when no alias changed | 1Shad0w/Lichii |
| Zen 3 + PGO build | clang-cl, ThinLTO, Lichii's PGO profile and `-march=znver3` | release asset |

PR #337 (fence-flagged query reports) is left out on purpose. Upstream measured it taking Ocarina
from about 48 to 32 FPS at GPU accuracy Low.

### Ultrawide 21:9

Ultrawide is not built in. It comes from Ivan0xFF's mod,
**[craftycorvid/oot-mod](https://github.com/craftycorvid/oot-mod)** (MIT). This build adds one thing
to it: with the 3440x1440 version of the mod enabled, the hearts and magic sit at the left edge of
the screen and the item buttons at the right edge, instead of in the centred 16:9
([#831](https://github.com/zortos293/Lichii/pull/831), commit `e0053016c2`). Menus stay centred.

To set it up:
1. Import the mod's zip under **Properties → Add-Ons → Import Mod from ZIP**.
2. Set **Graphics → Aspect Ratio → Stretch to Window**.

---

## Installing

1. Extract the whole zip and run `lichii.exe`.
2. Optional: copy `recommended-config/user` next to `lichii.exe` to use the per-game settings this
   build was tested with: docked, GPU accuracy Low and 12 GB of emulated memory, without ultrawide
   or frame generation. See `recommended-config/LEIA-ME.txt`.

There are two builds:

| File | CPU |
| --- | --- |
| `lichii-oot-mario-windows-x86_64-msvc.zip` | Any x86-64 CPU |
| `lichii-oot-mario-windows-x86_64-znver3-pgo.zip` | Ryzen 5000 or newer. It is faster, but it doesn't start on older CPUs |

## Switches

Each new part can be turned off from `nx2_switches.txt` next to `lichii.exe`. Restart after
editing it.

| Line | Effect |
| --- | --- |
| `EDEN_NX2_WATER_FIX=0` | Uses the old texture binding for the water |
| `EDEN_NX2_ATTR_MASK=0` | Pipeline keys keep every vertex attribute |
| `EDEN_NX2_IMAGE_MAY_FAIL=0` | An image that doesn't fit is created without memory, as before |
| `EDEN_NX2_OOT_BANK_SKIP=0` | Uniform banks are uploaded again once a frame |
| `EDEN_OOT_UW_HUD_EDGES=0` | The HUD stays centred with the ultrawide mod |
| `EDEN_NX2_OOT_FPS_UNLOCK=1` | Applies "FPS Unlock (built-in)" |
| `EDEN_NX2_OOT_OCCLUSION_OFF=1` | Applies "Occlusion Off (built-in)" |

## Building

Build it like Lichii on Windows, with MSVC 2022, the Vulkan SDK and Qt 6.8.3
(`cmake -G Ninja -DCMAKE_BUILD_TYPE=Release`). The Zen 3 build uses clang-cl 22 with
`-flto=thin -fprofile-use=lichii-windows.profdata /clang:-march=znver3`.

## Credits and license

All of the emulation work belongs to the developers of Lichii, Eglantine, Eden, yuzu and Sudachi,
and to the authors of each PR above. Every commit on this branch names the PR or fork commit it
comes from.

GPL-3.0-or-later. See [LICENSE.txt](LICENSE.txt).
