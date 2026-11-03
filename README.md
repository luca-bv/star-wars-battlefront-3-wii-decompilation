# Star Wars Battlefront III (Wii) Recompilation and Decompilation

A reverse-engineering project for **Star Wars: Battlefront III**, the cancelled
Free Radical Design game, working from the Wii build **2.91120a**
(disc ID `RABAZZ`, built November 21, 2008).

There are two tracks. The **static recompilation** translates the Wii executable
ahead of time into a native PC/Mac program, so the game runs without an
emulator. It is the furthest along, and you can already play matches with it. The
**matching decompilation** rebuilds C/C++ source that compiles to byte-identical
machine code for the original `main.dol`.

This repository is the public home page for the project. The working source
is not published yet. Watch or star the repo to hear when it is.

## Current state (November 2026)

### Recompilation

#### How it works

The recompiler, built on the open-source
[wiikit](https://github.com/vs-sr-dev/wiikit) toolchain, turns the game's
1.59 million PowerPC instructions (about 30,000 functions) into C++. Clang
compiles that into a normal executable, linked against a runtime that stands in
for the Wii's GPU, disc drive, IOS, memory and controllers. Only 119 words
could not be translated, all in exception vectors and low-level OS routines
that the runtime replaces. The decompilation's symbol map and its 217 typed
jump tables let the translator resolve switch statements it would otherwise
lose.

#### What runs

The game boots through the title, profile creation and menus
into Instant Action, and levels including Hoth, Yavin 4, Tatooine and Endor
run. On an Apple M1, Hoth and Yavin hold the game's 60 fps cap with the
renderer idle most of the frame. Endor runs at about 43 fps. A 15-minute session kept memory flat at 370-440 MB. Dying or restarting
a mission reloads the level. That used to hang until the port gave the game
the 128 MiB of MEM2 a development kit has, which this debug build expects.

#### Rendering

Vulkan is the default renderer (MoltenVK on macOS), with OpenGL
4.1 as a fallback. Recorded frames from the two differ in 0.002% of pixels. Shader
pipelines are cached between runs and built in the background at boot. A
texture cache tracks writes to guest memory, so textures are only re-hashed
when they change: Hoth went from 94.7 GB of hashing to 3.8 GB over 200 seconds.
Display lists keep their decoded vertices between frames, which took Endor
from 37 to 43 fps.

#### Controls

Keyboard, mouse and gamepad map onto the Wii Remote and Nunchuk
layout the game uses. The Wii executable also still contains a leftover
**PC control scheme** (71 bindings: WASD, mouse look, Esc for pause), and the
port can switch the game over to it. Wii Remote motions (twist steering in
vehicles, lightsaber slices) are not emulated yet.

#### Bugs found along the way

The game marks a
disc read as pending only after starting it, so a reply that arrives too fast
is lost and the loading screen never finishes. A real console is slow enough
that this never happens, and the runtime now holds replies back the same way.
Two visual bugs are in the original build itself. The sky dome is flat in every
level, because no Wii code sets the direction the dome is turned by. Some cloth
(capes, robes) sits about 1.9 m too high, because the game never writes the
cloth's fixed vertices and some models store them with an offset. The port fixes both, and each fix has a
switch that restores the original look.

#### Online play

The Wii build kept the full client/server netcode but had its
sockets removed: every send became a byte count and the receive path was
deleted. The port adds a UDP transport underneath the game's own code and passes
in the game's original command-line options (`-dedicatedserver`, `-client -ip`).
A client connects to a dedicated server and its first messages are
acknowledged. Then the server stops ticking its network manager. Getting a
client into a running match is the next milestone.

#### Still to do

A Windows build, motion controls, audio testing, online
matches, and removing the occasional stall the first time a shader combination
appears. The runtime changes are kept as 38 patches against wiikit, and the
general fixes will go upstream.

### Decompilation

| | Verified |
|---|---|
| Functions | 486 of about 29,600 |
| Code | 82,444 of 6,381,920 bytes (1.3%) |
| Data | 46,729 bytes |
| Source units | 386 |

Every accepted function is checked by a tool. It has to match the original
instructions, relocations and resolved bytes, link from its own source object,
and leave the full executable identical.

Reconstructed so far: the engine and platform memory allocators (heaps, arenas,
pools, handle memory), hash and named tables, the property system, the resource
I/O queue and path lookup, a complete zlib-style inflate decoder, DVD streaming
and its priority queue, thread and mutex helpers, and the C runtime's memory,
string, sort and formatting routines.

The next targets are the main allocator, matrix construction and transforms,
packaged resource reads, and the remaining table and pool initialization paths.
Gameplay code comes after that shared foundation.

The build is unfinished, so the decompilation keeps its bugs and no-op code
paths as they are. Fixes go in the port instead.

## FAQ

**Can I download and play it?** No. Nothing playable is released, and this
project will never distribute game code, data or assets. Recompiled code is
still the original game's code, so releases will not include a prebuilt
executable either. The plan is a launcher that checks your game files against
known hashes and builds the port on your machine.

**Is this a remaster or a mod?** No. The goal is to run the game as it was
left. The few fixes so far (sky, cloth) are optional and off with one switch.

**What platforms?** macOS today. Windows is planned.

**Can I help?** Contributor docs, build instructions and work claims will be
published with the source. Until then, open an issue if you have relevant
experience (PowerPC, CodeWarrior matching, Wii internals, Vulkan) or knowledge
of this build.

## Credits and legal

Star Wars: Battlefront III is the property of its respective owners. This is an
unaffiliated fan preservation project. No original code, binaries or assets are
hosted here. The recompilation is built on wiikit.
