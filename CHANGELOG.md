# Changelog

## 0.7.0 - Factory API
- **New:** native API 4 (`MSCore_api()` returns 4): typed command send (`MSCore_send_typed`) and factory senders: `find_buildable`, `build_place`, `disassemble`, `set_recipe`, `port_toggle`, `inv_move`. Lua: `cmd.send_typed`, `MSCore.factory.native`.
- **New:** `MSCore.factory`: footprints, placement check, machine config encoding, recipes.
- **New:** `MSCore.inv`: inventory slots, free space, stack moves.
- **New:** `MSCore.wbp`: UMG window helpers and geometry log.
- **New:** `MSCore.blocks` (Lua API 3): add a block to the building registry at runtime (`register`, `status`, `debug_dump`), so mods need no `DAT_BuildingBlockRegistry` pak. Research unlock is opt-in (`MSCore.blocks.allow_research = true`; every player needs the mod).
- **API numbers:** Lua API (`MSCore.api`, checked by `MSCore.require(n)`) is 3. Native API (`MSCore_api()`) is 4. They are separate counters. Module version 0.7.0.
- **Docs:** UI window lessons, pak patching guide (`mod-authors/patching-paks.md`), MSCore.wbp API.
- **Fixed:** scan cache: a cached row is dropped when its object is gone, and cached rows are at most 5 s old.
- **Fixed:** Lua scan fallback returns the same rows as the native scan (type filter, ship order and piloted fields, solar system id).
- **Fixed:** chunk hold: a chunk skipped while the player was in its zone is now loaded on a later hold / release call (was a permanent hole).
- Not yet run in game: the factory native senders and the API 4 exports. Lua syntax checked only.
- Not yet run in game: `MSCore.blocks` (registry append is unproven in game; research append is opt-in).

## 0.6.0 - Scan and chunk hold
- **New:** world scan of ships, resource asteroids and stations (`MSCore.scan`, native `MSCore_scan`, Lua `MSCoreNative_scan`). Cached per kind.
- **New:** chunk hold: keep the 3x3x3 chunk cube around a point loaded for an owner (`MSCore.chunks.hold` / `release`, native `MSCore_chunk_hold` / `MSCore_chunk_release` / `MSCore_chunk_ready`).
- **New:** selfcheck checks `scan` and `chunks`, and `scan.api`.
- Native API 3 (was 2). Lua API still 1. Scan and chunk hold are not verified in game yet.

## 0.5.1
- **Fixed:** update checks could show an old version because GitHub caches release files. Checks now always fetch the current manifest.
- **Removed:** `force_install`. Nothing is downloaded or installed unless you click Update / Add / Install. The game only checks and logs "update available".
- Status lines from the in-game loader no longer have a "force" field.
- Manager: Force install column removed.

## 0.5.0 - Mod loader
- **New:** MSCore Manager, a desktop app that finds the game, installs UE4SS + MSCore, and installs / updates / rolls back / removes mods.
- **New:** Add mod from `owner/repo` or any GitHub link.
- **New:** Remove mod removes everything the mod installed (files, UI, paks) and restores originals it replaced.
- **New:** in-game update check at start (log + `MSCore.loader.status()` / `updates()`).
- **New:** SHA-256 + size verification, zip path/type checks, one backup version for rollback.
- Public releases on GitHub (binaries only).

## 0.4.0 - Orders, warp and docks
- Shared native API v2: orders, autopilot, skip-to, filters, warp teleport, arrival and jump cost, sector list, map pin, dock and item lookup.
- Lua API: `orders`, `filters`, `warp`, `docks`, `find_item`.

## 0.3.0 - Ships, commands, steering
- `ships`, `cmd` and `steer` modules (list ships, send commands, aim and throttle).

## 0.2.0 / 0.1.0
- Core Lua helpers: safe object access, UI helpers, tick events, game name resolver, self-check log.
