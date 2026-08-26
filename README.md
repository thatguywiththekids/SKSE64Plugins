# skee64, the dll part of RaceMenu (fork)

This is a fork of expired6978/SKSEPlugins. Its main attraction is skee64, the dll part of the RaceMenu mod for Skyrim SE/AE. The fork exists since the upstream at the time of writing does not support the GOG version of the game (1.6.1179).

The skee64.dll produced by building this fork has been tested on 1.6.1179 and did not crash for me yet.

To build it, use Visual Studio on Windows or cmake on Linux with wine and an msvc cross build environment. On Arch Linux and its descendants there is an AUR package msvc-wine-git that helps.
