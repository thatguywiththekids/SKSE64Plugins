# skee64, the dll part of RaceMenu (fork)

This is a fork of expired6978/SKSEPlugins. Its main attraction is skee64, the dll part of the RaceMenu mod for Skyrim SE/AE. The fork exists since the upstream at the time of writing does not support the GOG version of the game (1.6.1179).

The skee64.dll produced by building this fork has been tested on 1.6.1179 and did not crash for me yet.

## Installation

This fork has no pre-built dll file, so you need to build it yourself.

To build it, one of the following setups can be used:

 * Visual Studio on Windows
 * cmake on Linux with wine and an MSVC cross build environment.

The SKSE source is also needed to build skee64. They are available in the SKSE downloads from skse.silverlock.org. Make sure to use the matching version, so you use SKSE for 1.6.1179 if you are building for GOG.

### Building on Windows

If you are used to Visual Studio, you probably know it better than me. Just keep in mind that the SKSE source tree needs to be placed as a sibling to SKSE64Plugins so this project can find what it needs.

### Building on Linux

For the time being I can only provide guidance for Arch Linux and its descendants like EndeavourOS.

The AUR package msvc-wine-git provides the necessary files for compiling Windows code.

To get to the SKSE source, you first need to unpack the downloaded 7z file (if you installed SKSE to Skyrim that should already be done). Then unpack `skse64_2_02_06_gog/src/skse64-2.2.6-gog.tar.gz` next to the `SKSE64Plugins` directory:

```bash
cd /path/to/SKSE64Plugins/..

# Unpack and rename to skse64
tar -xf /path/to/skse64-2.2.6-gog.tar.gz
mv skse64-2.2.6-gog skse64
```

Now you can configure and build using CMake:

```bash
cd SKSE64Plugins
cmake -G Ninja --toolchain=/opt/msvc/cmake/toolchain-x64.cmake -B build -S . -DSTORE_VERSION=1 -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

*(Note: Set `-DSTORE_VERSION=1` for GOG, or `-DSTORE_VERSION=0` for Steam.)*

After building, the resulting `skee64.dll` will be in `build/skee64/`. Where to put it depends on your Skyrim setup. Check where the skee64.dll from the RaceMenu mod has been installed and replace that in a way that fits your mod manager.
