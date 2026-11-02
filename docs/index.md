---
title: Home
nav_order: 1
layout: default
---

# MSCore

MSCore is a shared base for mods of the **Main Sequence Demo** game. It has three parts:

1. **Mod manager** (`MSCoreManager.exe`): installs MSCore and UE4SS, adds mods from GitHub, checks for updates, rolls back and removes mods.
2. **Lua helpers** (`mscore.lua`): safe wrappers around the game's objects, UI helpers, a 250 ms tick loop, ship and dock lookups, and command helpers for mods.
3. **Native part** (`MSCore.dll`, C++): sends ship commands, orders, warp and filter commands, and exports a C API for other native mods.

Current version: **0.5.1** (Lua API 1, native API 2).

## Who this is for

- **Players**: [Getting started]({{ site.baseurl }}/players/getting-started.html), [Safety]({{ site.baseurl }}/players/safety.html), [Troubleshooting]({{ site.baseurl }}/players/troubleshooting.html).
- **Mod authors**: [Quickstart]({{ site.baseurl }}/mod-authors/quickstart.html), [Lua API reference]({{ site.baseurl }}/mod-authors/lua-api.html), [Native C API reference]({{ site.baseurl }}/mod-authors/native-api.html), [Publishing a mod]({{ site.baseurl }}/mod-authors/publishing.html).
- **History**: [Changelog]({{ site.baseurl }}/changelog.html).

## Downloads are binaries only

The GitHub releases contain compiled files only: `MSCoreManager.exe`, the MSCore zip, the UE4SS zip and `manifest.txt`. No source code is published in the releases.

## Status markers

Some parts were built and tested only offline. Those parts are marked **Not verified in game** on each page. A mark means the code exists and was checked outside the game. It does not mean the feature is broken.

## Requirements

- Main Sequence Demo (Steam install).
- Windows.
- Internet access for the manager and for update checks (GitHub only).
