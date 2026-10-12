# KosmicKrisp pre-M5 compat ICD

Custom [Mesa 26.2.4](https://docs.mesa3d.org/drivers/kosmickrisp.html) **KosmicKrisp** Vulkan ICD for Apple Silicon (macOS 26+), meant as a Sikarugir drop-in.

From [MacGameTest](https://www.youtube.com/@MacGameTest) (`@truwa`).

**DOOM (2016)** already runs out of the box with current Sikarugir KosmicKrisp on **M5** (and should on **M6**). This build is for **pre-M5** chips (**M1–M4**), where stock `vkCreateDevice` fails because idTech enables `depthBounds` without querying.

This is **not** extra Metal functionality. It only **advertises** a few Vulkan 1.0 bits so those games can create a device.

| Feature | Upstream 26.2.4 | This build |
|---|---|---|
| `shaderCullDistance` | already `true` | unchanged |
| `depthBounds` | `false` | **`true` on all Apple GPU families** |
| `fillModeNonSolid` | `false` | **`true`** (Wolfenstein II enables wireframe without querying) |

**`depthBounds` is not wired to Metal.** A real depth-bounds test exists on Apple GPU family 10+ (M5). Advertising the bit *and* calling `setDepthTestMinBound` made Wolfenstein II lighting worse, so this ICD reports the feature and does not apply the test. DOOM 2016 does not need the actual test.

## Tested games (Sikarugir, native Vulkan)

| Game | Stock Sikarugir KK | This ICD |
|---|---|---|
| **DOOM (2016)** | Works on **M5** (expected on **M6**) | Needed on **M1–M4** so the game launches |
| **Wolfenstein II: The New Colossus** | `vkCreateDevice` failed (`fillModeNonSolid`) even on M5 | Launches on M4 and M5. Characters/cutscenes look fine. **World geometry (floors, ceilings, sometimes walls) flickers / renders wrong.** Same scene was OK on a patched MoltenVK. |

## Install (Sikarugir)

1. Download `libvulkan_kosmickrisp.dylib` from [Releases](https://github.com/truwa/kosmickrisp-compat/releases).
2. Quit Steam / the wrapper.
3. Back up, then replace `YourWrapper.app/Contents/Frameworks/libvulkan_kosmickrisp.dylib`.

## Optional error dump

By default this build **does not** write a log on the Desktop.

```sh
export KOSMICKRISP_DUMP_LOG=1
# or
export KOSMICKRISP_DUMP_LOG=/tmp/kosmickrisp.log
```

Set it where Wine actually inherits environment (wrapper `LSEnvironment` / CLI env), not only in a startup script subshell.

## Source

Patches against Mesa **26.2.4** are in [`patches/`](patches/). They are MIT, same as Mesa.

```sh
tar xf mesa-26.2.4.tar.xz
cd mesa-26.2.4
patch -p1 < ../patches/0001-kk-advertise-idtech-features.patch
patch -p1 < ../patches/0002-vk-log-optional-kosmickrisp-dump.patch
```

## License

MIT. KosmicKrisp / Mesa remain copyright of their authors (LunarG, Mesa, et al.). This repo only ships small compatibility patches plus a prebuilt ICD.
