# KosmicKrisp pre-M5 compat ICD

Custom [Mesa 26.2.4](https://docs.mesa3d.org/drivers/kosmickrisp.html) **KosmicKrisp** Vulkan ICD for Apple Silicon.

Built for **Sikarugir / Wine (x86_64, Rosetta)** on macOS 26+. It advertises a few Vulkan 1.0 feature bits that idTech enables without querying, so `vkCreateDevice` succeeds on **M1–M4** the same way stock KosmicKrisp does on **M5**.

From [MacGameTest](https://www.youtube.com/@MacGameTest) (`@truwa`).

## What this is (and is not)

This is **not** a full extra Vulkan feature implementation. It only **reports** bits so games that hard-require them at device create can launch.

| Feature | Upstream 26.2.4 | This build |
|---|---|---|
| `shaderCullDistance` | already `true` | unchanged |
| `depthBounds` | `false` | **`true` on all Apple GPU families** |
| `fillModeNonSolid` | `false` | **`true`** (Wolfenstein II also enables wireframe without querying) |

**`depthBounds` is not wired to Metal.** Metal 4 only has a real depth-bounds test on Apple GPU family 10+ (M5). Advertising the bit and calling `setDepthTestMinBound` made Wolfenstein II lighting worse, so this ICD lies about the feature and does not apply the test. DOOM 2016 does not need the actual test.

## Tested games (Sikarugir, native Vulkan)

| Game | Launch | Notes |
|---|---|---|
| **DOOM (2016)** | Works on M4 and M5 Pro | Playable with this ICD |
| **Wolfenstein II: The New Colossus** | Starts on M4 and M5 Pro | Characters/cutscenes look fine. **World geometry (floors, ceilings, sometimes walls) flickers / renders wrong.** Same scene was OK on a patched MoltenVK. That path is a KosmicKrisp gap, not a settings issue. |

## Install (Sikarugir)

1. Download `libvulkan_kosmickrisp.dylib` from [Releases](https://github.com/truwa/kosmickrisp-compat/releases).
2. Quit Steam / the wrapper.
3. Replace:

   `YourWrapper.app/Contents/Frameworks/libvulkan_kosmickrisp.dylib`

   Architecture must be **x86_64** (`file` should say `Mach-O 64-bit dynamically linked shared library x86_64`). An arm64 ICD will not load under Rosetta Wine.

4. Back up the original dylib first.

## Optional error dump

By default this build **does not** write a log on the Desktop.

```sh
# Desktop + /tmp
export KOSMICKRISP_DUMP_LOG=1

# or a specific file
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

Sikarugir needs an **x86_64** ICD (`-arch x86_64`, macOS 26 min). Native arm64 is a separate compile.

## License

MIT. KosmicKrisp / Mesa remain copyright of their authors (LunarG, Mesa, et al.). This repo only ships small compatibility patches plus a prebuilt ICD.
