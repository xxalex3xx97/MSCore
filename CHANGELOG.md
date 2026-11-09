# Changelog

## 0.8.2 - Events tick on the game thread

- **Fixed:** the events tick no longer runs Lua on UE4SS's async thread. That thread could corrupt the shared Lua state during long imports in other mods (Blueprint Unit). The tick now runs on the game thread.
- Native API stays 5, Lua API 3.
- **Not yet run in game:** the 0.8.2 fix. Checked by Lua syntax and code review only.

## 0.8.1 - Refund redirect only on demand

- **Fixed:** deconstructing something by hand put the refund into the Blueprint Unit instead of your inventory. The refund redirect was switched on from game start, so it caught every deconstruction in a station with a Blueprint Unit.
- **Changed:** the redirect is now off at game start. A mod that needs it calls `redirect_enable(true)` and must turn it off again (Blueprint Unit does this around its own builds, so its builds still take costs from the unit first). The file `Mods/MSCore/refund_redirect_on.txt` brings back the old always-on behaviour; `refund_redirect_off.txt` still blocks the start-time switch-on.
- **Confirmed in game:** hand deconstruction refunds go to your inventory, and Blueprint Unit builds still charge the unit first (2026-11-08).
- Native API stays 5, Lua API 3.

## 0.8.0 - Station block natives

- **New:** station block natives for Blueprint Unit's Station Build: `MSCore.native.add_block`, `remove_block`, `pending` (reset or apply the pending blueprint) and `read_block_class` (class data: bounds, cost, station block or not), with wrappers in `MSCore.factory_native`. C++ native API is now 5.
- **New:** `read_station` block lines carry the full class path as an optional 7th column (older 6-column lines still parse).
- **Fixed:** the click hook of MSCore windows is removed with a short delay and a reopen cancels the removal. Buttons the engine no longer knows are skipped instead of bound. This targets a crash that happened when the dev menu inventory panel was opened again.
- **Confirmed in game:** with Blueprint Unit, a Station 1x1 was placed where a Shipyard side stood (the side was removed first), 2026-11-08.
- **Not yet confirmed:** the click-hook crash fix (needs a few open/close rounds of the dev menu).
- **Known limits:** blocks of other classes and rotations were not tried in game.

## 0.7.6 - Local inventory split, camera block, Dock_2 floor

- **New:** native inventory split (`MSCore.native.inv_split`, `MSCore.factory_native.inv_split`, C export `MSCore_inv_split`). Moves an amount of items from one slot to an EMPTY slot of the same inventory. It is a LOCAL edit, so co-op clients do not see it: use it in single-player only. Co-op desync is accepted for this version. Needed by Blueprint Unit 0.2.0 (local inventory split).
- **New:** while an MSCore window is open the camera look is blocked. Closing the window restores the previous setting.
- **Fixed:** Dock_2 floor is 30x30, and a footprint across touching floors validates against the padded floor union.
- **Fixed:** binary config bytes containing a NUL were cut at the first NUL, so machine configs, recipes and ports never reached the game. Blueprint Unit 0.2.0 needs MSCore 0.7.6 for this.
- **Not yet run in game:** inv_split, the camera block, the Dock_2 floor. Lua tests only; the NUL fix is checked by code read.
- **Known limits:** inv_split is single-player only (see above). A WBP window destroyed without close (death, load) may leave the camera input setting until the next window open/close.

## 0.7.5 - Blueprint Unit refund and charge redirect

Includes everything since 0.7.0 (0.7.1 to 0.7.4 were not released on their own).

- **New (default ON):** with a Blueprint Unit container in a station, deconstruction refunds go into the unit first, then your inventory. Machine build costs are taken from the unit first, then from inventory. Turn off with the file `Mods/MSCore/refund_redirect_off.txt`. The redirect needs the Blueprint Unit container (`BP_BPUnitStoreObj_01_C`) to do anything.
- **Confirmed in game:** refund redirect works with Blueprint Unit (2026-11-07).
- **New:** `inv_move` takes an optional receiver (the player object id, Lua 5th argument, C export `MSCore_inv_move_to`). Calls without it still send receiver 0, which the game drops silently.
- **New:** native station read (`MSCore.native.read_station`, `MSCore.factory_native.read_station`) and native inventory read (`read_inventory`). `MSCore.inv.slots` uses the native read when present.
- **Fixed:** the `MSCoreNative_*` Lua globals (`MSCore.native`) are registered in every Lua mod, not only MSCore. Other mods can use `cmd.send` and the factory natives.
- **Experimental, off by default:** refund probe (log only, `refund_probe.txt`). It logs what the redirect would do.
- **Known limitation, co-op:** the game is lockstep. Every player needs 0.7.5 with the redirect on and the Blueprint Unit container pak, or refunds and charges land in different inventories and the game desyncs. Designer (blueprint) and teardown paths are untested.
- **Not yet run in game (standalone):** native station read, native inventory read, the lua globals fix, the probe. Lua syntax and the fake-function harness only.
- **Blueprint Unit (separate mod, not part of this release):** container contents restore is parked and experimental, off by default (`Mods/BlueprintUnit/restore_on.txt`).

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
