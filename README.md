<div align="center">

# MSCore

### Mod loader, updater and shared library for *Main Sequence*

[![Latest release](https://img.shields.io/github/v/release/xxalex3xx97/MSCore?style=for-the-badge&label=release)](https://github.com/xxalex3xx97/MSCore/releases/latest)
![Platform](https://img.shields.io/badge/platform-Windows-blue?style=for-the-badge)
![Game](https://img.shields.io/badge/game-Main%20Sequence%20Demo-orange?style=for-the-badge)
[![Docs](https://img.shields.io/badge/docs-website-brightgreen?style=for-the-badge)](https://xxalex3xx97.github.io/MSCore/)

**Install mods, keep them up to date, and remove them cleanly - from one small window.**

</div>

---

**📖 Documentation: https://xxalex3xx97.github.io/MSCore/** (player guide, Lua API, native API, publishing a mod)

## What is MSCore?

MSCore is two things in one download:

| | What it does |
|---|---|
| **MSCore Manager** (`MSCoreManager.exe`) | A desktop app. Finds your game, installs the mod framework (UE4SS) and MSCore, then installs, updates and removes mods straight from GitHub. |
| **MSCore (in game)** | A library mods load on top of. It gives them safe, shared building blocks for ships, orders, docking, warping and more, so every mod does not have to reinvent them. It also checks for mod updates when the game starts. |

You do not need to know anything about modding to use it.

## Quick start

1. Close the game.
2. Download **`MSCoreManager.exe`** from the [latest release](https://github.com/xxalex3xx97/MSCore/releases/latest) and run it.
3. It finds your Main Sequence folder automatically (Steam). If it cannot, it asks you to pick the folder.
4. Click **Install/Repair MSCore**. This installs UE4SS and MSCore and puts MSCore first in the mod list.
5. Click **Add mod...** and paste a mod's GitHub address, for example
   `https://github.com/xxalex3xx97/MiningBenchMod` (or just `xxalex3xx97/MiningBenchMod`).
6. Start the game from Steam.

> Updates and installs only ever happen when **you** click a button. Nothing is downloaded in the background.

## The manager

| Button | What it does |
|---|---|
| **Check** | Looks up the newest release of every installed mod. |
| **Add mod...** | Installs a mod from a GitHub address (`owner/repo` or a full GitHub link). |
| **Update selected / Update all** | Downloads and installs newer versions. They are applied safely and the old files are kept as a backup. |
| **Install/Repair MSCore** | Installs or fixes UE4SS + MSCore. |
| **Rollback** | Puts the previous version of the selected mod back. |
| **Remove mod** | Removes a mod and everything it installed (its files, UI and packs). Your own settings and logs are left alone. |
| **Browse game folder** | Choose the game folder by hand. |

The list shows each mod's installed version, the latest version, and its status.

## Safety

- Downloads come only from **GitHub** over HTTPS.
- Every download is checked against a **SHA-256** hash and size from the release manifest before anything is installed.
- Zip files are inspected first: no paths outside the game's mod/pak folders, and only known file types (`.dll .lua .pak .utoc .ucas .txt .mod .ini`).
- Your settings (`MiningBench.ini`, `loader.ini`) and logs are never overwritten.
- One previous version is kept so you can **Rollback**.
- The game only **checks** for updates on start and tells you in the log. It never downloads by itself.

A hash proves a file was not damaged or swapped in transit. It does not prove a mod is safe: only add mods from authors you trust.

## For mod authors

A mod becomes manageable by shipping a `Mods/<Name>/mscore.mod` file and a GitHub release:

```ini
name=MyMod
repo=owner/MyMod
version=1.0.0
```

Release assets: a `manifest.txt` and the zip it names.

```ini
name=MyMod
version=1.0.0
zip=MyMod-1.0.0.zip
sha256=<hash of the zip>
size=<bytes>
min_mscore_native=0.5.0
```

Zip layout: `Mods/<Name>/...` goes to the game's UE4SS `Mods` folder, `Paks/...` to the game's `Paks` folder. Tag the release `v<version>`.

In game, mods can use the MSCore Lua API (`MSCore.ships`, `orders`, `warp`, `docks`, `loader`, ...) and native exports (`MSCore_api()` returns 2).

## Files in each release

| File | Purpose |
|---|---|
| `MSCoreManager.exe` | The manager app. |
| `MSCore-<version>.zip` | MSCore itself (used by the manager). |
| `ue4ss-<version>.zip` | The UE4SS framework build the game needs (MIT licensed, license included). |
| `manifest.txt` | Version and checksums the manager reads. |

## Credits and license

MSCore bundles a build of [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) (MIT license, included in the download). *Main Sequence* is the property of its developers; this project is an unofficial community tool.

See [CHANGELOG.md](CHANGELOG.md) for what changed in each version.
