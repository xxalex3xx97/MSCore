---
title: Lua API reference
parent: Mod authors
nav_order: 2
---

# Lua API reference

Lua API version: **1**. Module version: **0.5.1**. Load it with `require("mscore")` after the path header in the [Quickstart]({{ site.baseurl }}/mod-authors/quickstart.html).

Conventions:

- `ok, err` return pairs: `ok` is `true` or `false`, `err` is a string. When there is no native DLL, every native-backed function returns `false, "native DLL not loaded"`.
- Coordinates in `ch` (chunk) are integers. Locations in `loc`/`arr` are raw fixed-point numbers. Divide by `units.RAW_PER_M` to get metres.
- Read-only lookups use a 5 second cache unless noted.

Each section names the source file. All names below are in `mods/MSCore/Scripts/mscore.lua` unless noted.

## MSCore (top level)

| Name | Type | Notes |
|---|---|---|
| `MSCore.api` | number | `1` |
| `MSCore.version` | string | `"0.5.1"` |
| `MSCore.verified_build` | number | `149200896`, the game exe size MSCore was verified with |
| `MSCore.native` | table or `nil` | Wrappers for the `MSCoreNative_*` globals. `nil` when the DLL is not loaded. |

### MSCore.require(n)

Errors when `n` is not a number or is larger than `MSCore.api`. Returns `MSCore`.

```lua
MSCore.require(1)
```

### MSCore.logger(tag)

Returns `log, try`.

- `log(...)` prints `tag` followed by all arguments joined with spaces.
- `try(label, f, ...)` calls `f` with `pcall`, logs `label ok` or `label FAIL: <error>`, and returns `ok, result`.

```lua
local log, try = MSCore.logger("[MyMod] ")
local ok, v = try("read hull", function() return 1 + 1 end)
```

## MSCore.resolve

Table of game class and path names. Read these instead of writing your own strings.

| Field | Value |
|---|---|
| `player` | `{ "DMSPlayerObject", "BP_PlayerObject_C" }` |
| `controller` | `{ "DMSPlayerController", "BP_PlayerController_C" }` |
| `controllerPath` | `"/Script/MainSequence.DMSPlayerController"` |
| `invWidget` | `"WBP_CharacterStatus_C"` |
| `missionsWidget` | `"WBP_MissionsPanel_C"` |
| `item` | `"Item"` |
| `research` | `"ResearchTechnology"` |
| `umg` | `"/Script/UMG."` |
| `clickHook` | `"/Script/UMG.Widget:ForceLayoutPrepass"` |
| `ship` | `{ "DMSPlayerShipObject" }` |
| `console` | `"PilotConsoleObject"` |
| `benchBlocks` | `{ "BPO_MiningBenchBlock_C", "BPO_MinerModuleBlock_C" }` |
| `shipProps` | `{ id = "ID", dock = "Dock", dockId = "DockID", maxHull = "MaxHealth" }` |
| `dock` | `"Dock"` |
| `sector` | `"SolarSystemSector"` |
| `hullFns` | `{ "GetCurrentHull" }` |

`player`, `controller`, `ship` are lists of candidate names. Pass them to `safe.live` with `table.unpack`.

## safe

| Function | Returns | Notes |
|---|---|---|
| `safe.try(label, f, ...)` | `ok, result` | Same as the `try` from `logger`. |
| `safe.val(v)` | value | For userdata, tries `v:get()` then `v:Get()`. Other values are returned unchanged. |
| `safe.nameOf(x)` | string | `x:ToString()`, or `tostring(x)` on failure. |
| `safe.valid(o)` | boolean | `true` only when `o` is userdata and `o:IsValid()` is `true`. Call before any method on a game object. |
| `safe.objName(o)` | string | Object name, or `"?"` when `o` is not valid. |
| `safe.live(...)` | object or `nil` | First valid hit of `FindFirstOf` over the class names given. No logging. Safe to call every tick. |
| `safe.rd(f)` | string | `tostring(f())`, or `"?"` when `f` errors or returns `nil`. |

```lua
local hull = MSCore.safe.rd(function() return ship:GetCurrentHull() end)
```

## units

Status: **Not verified in game** (no consumer yet).

| Field or function | Notes |
|---|---|
| `units.RAW_PER_M` | `65536` raw units per metre |
| `units.CHUNK_M` | `2048` metres per chunk |
| `units.abs_pos(chunk, loc)` | `chunk` = `{X,Y,Z}` integers, `loc` = `{X,Y,Z}` raw numbers. Returns `{X,Y,Z}` in metres. |
| `units.dist_m(a, b)` | Straight-line distance between two `{X,Y,Z}` points, in metres. |

```lua
local p = MSCore.units.abs_pos({ X = 1, Y = 0, Z = 0 }, { X = 65536, Y = 0, Z = 0 })  -- X = 2049
```

## ui

| Function | Notes |
|---|---|
| `ui.mk(cls, outer)` | Creates a UMG object from `/Script/UMG.<cls>` (for example `"TextBlock"`), owned by `outer`. |
| `ui.setText(tb, s)` | Sets a TextBlock's text. |
| `ui.onClick(btn, fn)` | Calls `fn()` on the game thread when `btn` is clicked. |
| `ui.lin(c)` | Converts one sRGB value (0 to 255) to a linear 0 to 1 number. |
| `ui.rgba(r, g, b, a)` | Returns `{ R, G, B, A }` with `r, g, b` converted by `lin`. `a` defaults to `1`. |
| `ui.smallFont(tb, size)` | Sets the font size of a text block. Logs one failure line at most. |
| `ui.hadd(hb, w)` | Adds `w` to a horizontal box with 4 px left and right padding. |

```lua
local tb = MSCore.ui.mk("TextBlock", controller)
MSCore.ui.setText(tb, "Hello")
local red = MSCore.ui.rgba(255, 0, 0)
```

The click handler is registered through `ForceLayoutPrepass`. This is a workaround documented in `hacks` (`click_hook`).

## events

| Function | Returns | Notes |
|---|---|---|
| `events.on("tick", fn)` | id | Runs `fn()` every 250 ms on the game thread. Only `"tick"` is accepted in API 1. Any other name raises an error. |
| `events.off(id)` | nothing | Removes the callback. |
| `events.running()` | boolean | `true` once the shared loop has started. |

All `events.on` callbacks in one Lua state share one loop. Each callback runs under `pcall`, so an error in one does not stop the others.

## loader

Status: the Lua functions read rows from the native part. The update check runs in a background thread of the DLL. **Not verified in game** for the in-game notice path. The manager update flow was tested.

| Function | Returns | Notes |
|---|---|---|
| `loader.status()` | list of rows | Row = `{ name, installed, remote, state }`. Empty list when the scan has not run or the DLL is missing. |
| `loader.updates()` | list of rows | Only rows where `state == "update"`. |
| `loader.on_update(fn)` | nothing | Calls `fn(row)` once per mod with an update. Fires on the first tick after every mod has been checked. |

`state` is one of `pending`, `uptodate`, `update`, `skipped:min_native`, or `error:<msg>`. The DLL only checks. It never downloads.

```lua
MSCore.loader.on_update(function(row)
    print("update for " .. row.name .. ": " .. row.installed .. " -> " .. row.remote .. "\n")
end)
```

## hacks

| Function | Returns | Notes |
|---|---|---|
| `hacks.add{name, what, why, verified_build}` | the table | Registers a workaround in this Lua state. |
| `hacks.list()` | list | All registered workarounds. |

MSCore registers six entries: `click_hook`, `valid_before_call`, `linear_colors`, `shipmove_typeid`, `map_pin_hook`, `teleport_warp`.

## resolve

See the `MSCore.resolve` table above.

## ships

Read-only. The ship list is cached for 5 seconds. Every entry is checked with `safe.valid` before use. Status: **Not verified in game** for this module.

| Function | Returns | Notes |
|---|---|---|
| `ships.list()` | list of `{ id, name, obj }` | Every ship. `name` is the display name, or `"?"`. |
| `ships.mine()` | list of `{ id, name, obj }` | Ships with a `BPO_MiningBenchBlock_C` or `BPO_MinerModuleBlock_C` block. |
| `ships.get(id)` | entry or `nil` | Entry for one ship id. |
| `ships.is_docked(id)` | boolean, or `nil, reason` | `true` when the ship has a dock and a non-zero `DockID`. |
| `ships.hull(id)` | `{ current, max }`, or `nil, reason` | From `GetCurrentHull` and `MaxHealth`. A field is `nil` when unreadable. |
| `ships.pilot_console(id)` | object or `nil` | First `PilotConsoleObject` with `ShipID == id`, otherwise `ParentID == id`. |
| `ships.warp_fuel(id)` | number, or `nil, reason` | `GetAvailableJumpFuel`. Counts warp fuel items only (1 item = 50 km). Hydrogen and deuterium are not wrapped. |

```lua
for _, s in ipairs(MSCore.ships.mine()) do
    print(s.id .. " " .. s.name .. "\n")
end
```

## cmd

| Function | Returns | Notes |
|---|---|---|
| `cmd.send(receiver, cmdObj)` | `ok, err` | Sends `cmdObj` to `receiver` (a ship id) with `AddInputCommand`. Applies the type id fix (`shipmove_typeid`). Needs the native DLL. |

`cmdObj` is a UObject made with `NewObject` or a `CreateInput*` factory. The mining mod uses this path in game.

## steer

Needs the native DLL.

| Function | Returns | Notes |
|---|---|---|
| `steer.aim(id, yaw, pitch)` | `ok, err` | Sets the view offset in degrees through the ship's pilot console. Raw value is degrees times 65536. |
| `steer.stop(id)` | `ok, err` | Same as `aim(id, 0, 0)`. The throttle is not changed. |
| `steer.throttle(id, delta)` | `ok, err` | Sends a throttle step of `delta` (integer) to ship `id`. |

Status: **Not verified in game** for `aim` and `throttle` through MSCore. The Ctrl+F9 test key in `main.lua` was not confirmed.

## orders

Needs the native DLL. Every function returns `ok, err`.

| Function | Notes |
|---|---|
| `orders.new(receiver, cls)` | Creates an order of class `/Script/MainSequence.<cls>` for `receiver`. Returns `ok, err`. The new order id is not returned. |
| `orders.update(oid, target, item, cmp, amount)` | Updates order `oid`. `item` is a UObject or `nil`. `target`, `cmp`, `amount` are integers. |
| `orders.delete(sid, oid)` | Deletes top-level order `oid` of ship `sid`. |
| `orders.autopilot(sid, on)` | Turns the ship autopilot on (`true`) or off (`false`). |
| `orders.skip_to(sid, idx)` | Jumps to order index `idx`. Returns `ok, err, before, after` (the current order index before and after). |

Status: **Not verified in game** through MSCore (API 2).

## filters

| Function | Notes |
|---|---|
| `filters.send(sid, idx, item, amount, action, remove)` | Sets logistic filter row `idx` of ship `sid`. `item` is a UObject or `nil` (empty row). `action`: 0 = LIMIT, 1 = BUFFER. `remove = true` removes the row. Returns `ok, err`. |

Status: **Not verified in game** through MSCore (API 2).

## warp

Needs the native DLL.

| Function | Returns | Notes |
|---|---|---|
| `warp.send(sid, sys, ch, arr, wrap, set_target)` | `ok, err` | Teleport warp for ship `sid` into solar system `sys`. `ch` = chunk `{X,Y,Z}`, `arr` = raw arrival location `{X,Y,Z}`. `wrap` = wrap in piloting on/off. `set_target` = set the jump target. |
| `warp.pinned_sector()` | number | Last map-pinned sector id. `0` when none or no DLL. |
| `warp.sectors()` | list | Every `SolarSystemSector`, cached 5 s. Each entry: `id`, `sys`, `ch` (`{X,Y,Z}`), `he` (half extents `{X,Y,Z}`), `name`, `warp_dist` (or `nil`), `ores` (list of item UObjects). |

The warp follows the manual warp the player sends: `SetPilotingInput` on, `ShipTeleportInput`, `SetPilotingInput` off. An instant warp used no warp fuel in the MiningBench in-game test. The mining mod undocks a ship before it warps. MSCore does not check the dock state itself.

Status: **Not verified in game** through MSCore (API 2).

## docks

| Function | Returns | Notes |
|---|---|---|
| `MSCore.docks()` | list of `{ id, name, obj }` | Every dock. Not cached: each call runs `FindAllOf`. Call only on a user action. |

Status: **Not verified in game** through MSCore.

## find_item

| Function | Returns | Notes |
|---|---|---|
| `MSCore.find_item(res)` | item object, or `nil, why` | Matches the object name, the display name (case-insensitive) or a numeric ID. Zero matches or several matches return `nil` with the reason. The item list is cached 5 s. |

```lua
local iron, why = MSCore.find_item("Iron")
if not iron then print(why .. "\n") end
```

Status: **Not verified in game** through MSCore.

## native

`MSCore.native` is `nil` when the DLL is not loaded. Otherwise it has these fields. Each one is the raw `MSCoreNative_*` global.

| Field | Lua signature | Returns |
|---|---|---|
| `version` | `()` | string, the native version (`"0.5.1"`) |
| `alive` | `(obj)` | boolean. `false` for `nil`, numbers and dead objects. |
| `send_cmd` | `(receiver, cmd)` | `ok, err` |
| `view_send` | `(cid, yawDeg, pitchDeg)` | `ok, err` |
| `throttle` | `(sid, delta)` | `ok, err` |
| `offsets` | `()` | string: `typeid_off=0x28 throttle_id=0x3a view_raw_per_deg=65536` |
| `check_offsets` | `()` | `ok, detail` |
| `new_order` | `(receiver, cls)` | `ok, err` |
| `update_order` | `(oid, target, item, cmp, amount)` | `ok, err` |
| `delete_order` | `(sid, oid)` | `ok, err` |
| `autopilot` | `(sid, on)` where `on` is `0` or `1` | `ok, err` |
| `skip_to` | `(sid, idx)` | `ok, err, before, after` |
| `filter_send` | `(sid, idx, item, amount, action, remove)` | `ok, err` |
| `warp_send` | `(sid, sys, cx, cy, cz, ax, ay, az, wrap, set_target)` | `ok, err` |
| `pinned` | `()` | sector id, `0` = none |
| `pin_ensure` | `()` | `true` once the map pin hook is armed. Call every tick until it returns `true`. |
| `check` | `(name)` | `ok, detail`. Read-only world checks. Names: `classes`, `controller`, `sectors`, `arrival`, `jump_cost`, `pin_hook`, `filters_read`, `find_item`, `find_dock`. |
| `loader_status` | `()` | string, one line per mod: `name\|installed\|remote\|state` |

`autopilot` takes the number form directly. The Lua `orders.autopilot` converts a boolean for you.
