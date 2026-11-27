# Changelog

## 0.9.3 - Rock registry fix

- **Fixed:** the rock registry world could fail to load in sessions that start while piloting a ship (a stale menu controller was kept), so rock-dependent features stayed idle. It now looks the controller up again until it has a live player. Not yet run in game.
- **Fixed:** the manager's self-update is also handed back when an install or remove finishes, not only after the startup check.
- **Added:** the log now reports why the rock registry world is not loading (at most every 10 s).

## 0.9.2 - Manager auto-update fix

- **Fixed:** the manager's startup self-update was silently dropped when it arrived while the startup mod check was still running, so the manager never updated itself by itself. It now waits for the check to finish and then updates. Tested: the GUI replaced a 0.9.0-labelled build with the released manager and restarted.
- Managers 0.9.0 and 0.9.1 still have this bug: update them once by hand (download `MSCoreManager.exe` from this release, or run `MSCoreManager.exe --self-update`). From 0.9.2 on it updates itself.

## 0.9.1 - Remove mod fix

- **Fixed:** Remove mod refused or could leave broken references in some saves (stray modded class entries inside embedded designs, or all spare classes already used). The modded class slot in the save now points at the stock class, so other references to it land on the stock class too.
- Tested in game on one save that failed before (loads, designer opens). Other saves are checked by the scrubber's own verification only.

## 0.9.0 - UE4SS 4.0.0

- **Changed:** MSCore and the manager now run on UE4SS 4.0.0. The release ships `ue4ss-4.0.0.zip` and the manager installs it. Tested in game by the author (selfcheck ok=97, broken=0).
- **New:** the manager and the in-game loader refuse to run on an older UE4SS, with a message that says which version is needed (`min_ue4ss`).
- **New (mods):** ballistics (`MSCore_ship_guns`, `MSCore_sim_rate`), enemy stations (`MSCore_enemy_stations`), floaters (`MSCore_floaters`), chunk stats (`MSCore_chunk_stats`) and dock kind (`MSCore_dock_kind`). Each module has its own kill switch.
- **Changed:** the chunk hold cap is removed (the real limit is what the game can load). `MSCore_enemy_velocity` and `MSCore_enemy_motion` now use the measured sim rate (speeds were about 2.7x too high).
- **Dev only:** DevEval, a runtime probe tool that stays off unless `Mods\MSCore\deveval_on.txt` exists.
- Native API 28 (was 16), Lua API 3 (additions only).
- Mods built for UE4SS 3.x may need an update for UE4SS 4.0.0.
- Not yet run in game: floaters and enemy stations.
## 0.8.15 - Holograms, fleets, rock registry, manager clean-up, designer fix

- **Fixed:** the game crashed with an access violation (0x110) when opening the ship or station design editor. MSCore now guards the game function behind it (`MSCore.designer`, own kill switch). Tested in game on ship and station designers.
- **New:** rock registry service (native API 16): one shared, fast list of asteroids for mods (`MSCore_rockreg_*`). Tested in game.
- **New:** ship holograms and item icons for mod windows, fleets (fleet window, add-to-fleet, native fleet read and commands), ship repair, station names, ship hull export and mining capability flags (native API 15).
- **Manager:** Remove now cleans up what a mod left behind (including files in other folders) and scrubs the mod's block classes from saves and designs, with a backup first, so the game no longer crashes on load. Disable moves the mod's files out of the game folders so the game really stops loading it. The manager shows its version in the title and a label, and updates itself (checkbox "Auto-update manager", `--self-update`).
- Native API 16, Lua API 3 (additions only).
- Not yet run in game: the manager self-update.

## 0.8.14 - Game update 2026-11-20

- **Works with the 2026-11-20 game update.** MSCore re-checked every game hook against the new game build: nothing needed a new pattern, and the self-check is all green (no false "BROKEN" for `rename_ship` any more).
- **New:** `world` module: MSCore can tell whether the game is paused (export `MSCore_world_paused`). Mining Bench Mod uses it so a long pause no longer makes a ship's trip fail with "clear timeout".
- Native API 10, Lua API 3 (additions only). Verified in game: self-check ok=80, broken=0, and a 7-ship park-and-pause test.

## 0.8.13 - Flight module and required mods

- **New:** MSCore Manager installs required mods automatically. A mod can list the mods it needs (`requires=` in its manifest); Add and Update install the missing ones first, stop with a clear message if one is not available, and Remove warns which installed mods depend on the one you remove. A mod that lost a required mod shows up in the list and Update repairs it. New command line option `--update <ModName>`.
- **New:** `flight` module: one safe read of a ship's flight state (position and orientation, throttle and steps, autopilot flags, collision radius, power) and exports to set the throttle, zero the controls and find the pilot console. `COMPAT CAPS ... flight=ok` in the log, own kill switch. Used by ShipControl; game names for ship flight now live only in MSCore.
- Native API 10, Lua API 3 (additions only). Verified in game with Mining Bench Mod 0.3.0 and ShipControl 0.3.0 (miners approach, mine, reroute and brake as before).

## 0.8.12 - Ship shields

- **New:** a ship's current and max shield can be read through MSCore (`MSCore_ship_shields`), in its own `ships` module with a capability line `COMPAT CAPS ... ships=ok` in the log and a kill switch that turns only that module off.
- **New:** the ship class is looked up by logical name (`MSCore.class("ship")`), like asteroids and stations. After a game update only MSCore needs the fix.
- **New:** `factory_native.rename_ship(shipId, text)` renames a ship (Lua).
- Native API 9, Lua API 3 (additions only). The `ships` module self-test passes in game (`ships=ok`); a live shield value in a fight was not yet seen.

## 0.8.11 - Hook crash fix, class names and rock scan

- **Fixed:** a crash when a mod window and the dev menu were opened in turn. The layout hook is registered once and kept, instead of being removed and registered again on every open.
- **New (0.8.10):** the game class names for asteroids, stations, containers and the research block live in one table inside MSCore and are looked up by logical name (`MSCore.class`). Native rock scan (`MSCore_scan_rocks` / `MSCore.rocks`) with a `rocks` capability and a layout check that turns it off safely. `COMPAT CAPS ... rocks=ok` in the log.
- Native API 8, Lua API 3 (additions only). Run in game with the Mining Bench Mod and Blueprint Unit.

## 0.8.9 - Widgets by logical name

- **New:** game widgets (inventory grid, item amount, fleet ship entry, pause menu) are found by one logical name each, from a single table inside MSCore, then by name. If a game update renames one, only MSCore needs the fix. The Mining Unit window and the Blueprint Unit use it.
- **New:** `COMPAT CAPS widgets=ok` in the log, and a kill switch that turns only the widget module off.
- No API break: Lua API 3 (additions only), native API 7. Mod windows run in game with this build.

## 0.8.8 - Dock filters

- **New:** `MSCore.factory_native.set_dock_filter(receiverId, index, itemName, bLoading)` sets a dock's item filter the way the game's own dock window does. The Blueprint Unit 0.4.3 uses it to apply dock filters when you paste a station.
- No API change: native API 7, Lua API 3. Run in game through a Blueprint Unit paste.

## 0.8.7 - Chunk hold radius, safer modules

- **New:** C export `MSCore_chunk_hold2(owner, sys, ch3, radius, err, errlen)`. Radius 1 = 3x3x3 chunks (same as `MSCore_chunk_hold`), radius 0 = the centre chunk only.
- **New:** `MSCore.hook` in Lua: a RegisterHook wrapper that never throws and reports through a `COMPAT` capability line.
- **Changed:** every export is guarded: an error in one feature turns only that feature off (one `COMPAT` line names it) and never takes other modules or mods down. The refund redirect, command ids and renames read and write game memory through guarded accessors.
- **Changed:** command type ids now live in one place and their class layout is checked before the first use. If it does not match, commands turn off and the log says so instead of writing to a wrong place.
- **Fixed:** after a Blueprint Unit run the refund redirect no longer stays on: manual build and deconstruct refunds go back to your inventory 8 seconds after the last redirected item.
- Native API 7, Lua API 3. Refund redirect, idle drain and build charge run in game; chunk hold radius used in game with the Mining Bench Mod (no faults). The guarded command ids are not fully run in game yet.

## 0.8.6 - Update-proof addresses

- **Changed:** the refund redirect and the inventory split no longer depend on fixed game addresses or an exe size check. The functions they call, the group type and the player Inventory / creative-flag offsets are found when the game starts. If one cannot be found the feature turns off and the log says which (`COMPAT ...`), instead of running on a wrong address.
- **New:** one `COMPAT CAPS` line in the log lists what was found (chunk hold, refund; split and player offsets appear on first use).
- Refund redirect run in game on the current build; inventory split not yet.

## 0.8.5 - Game update compat

- **Fixed:** the refund redirect works again on the new game build (exe size 149299200). On the new build MSCore refused to start it ("COMPAT refund failed: exe size") and the Blueprint Unit fell back to its legacy build flow. The exe size check, the function addresses and the player creative-flag offset now follow the new game build.
- No API change. Not yet run in game beyond a Blueprint Unit build click.

## 0.8.4 - Quiet install

- **Changed:** a fresh install no longer shows the UE4SS console and logger windows. The `UE4SS-settings.ini` that ships with MSCore has `ConsoleEnabled`, `GuiConsoleEnabled` and `GuiConsoleVisible` set to `0`.
- The manager never overwrites an existing `UE4SS-settings.ini`. If you already have one, your console stays until you set those three lines to `0` yourself.
- No API change: native API 6, Lua API 3. Not yet run in game.

## 0.8.3 - Dock names

- **New:** `MSCore.factory_native.rename_dock(receiverId, text)` renames a dock, industrial dock or shipyard dock the same way the game's own rename window does. The name is built natively, never through Lua (a Lua version crashed the game).
- **New:** `MSCore.factory_native.read_docks(stationId)` lists the docks of one station as `{machine_id, block_id, parent_id, class, name, raw_id}`.
- **New:** the station read gives each block its BlockID (`bid`, `-1` when unreadable) and the MachineIDs of its child machines (`mids`). A dock's own BlockID can read `-1`, so match a block to its dock through `mids`.
- **New:** the self check reports the dock read and how many blocks link to a dock. It no longer walks game structures from Lua (a build of this release that did crashed the game; removed before release).
- **Confirmed in game:** the block-to-dock link matched 20 of 20 docks on a real station, and renaming docks works, including a dock placed by Station Build (2026-11-10).
- Native API 6 (the new calls are Lua only), Lua API 3.
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
