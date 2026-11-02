---
title: Native C API reference
parent: Mod authors
nav_order: 3
---

# Native C API reference

The native part is `MSCore.dll`, installed as `ue4ss\Mods\MSCore\dlls\main.dll`. It exports C functions that other native mods (C++ UE4SS mods) can call with `GetProcAddress`.

**Native API version: 2.** `MSCore_api()` returns `2`. Check it before you use the other exports. Use `MSCore_api` to decide if your mod can run. The MiningBenchMod refuses to load below 2.

Status markers:

- Exports marked **in game (MiningBench)** were used by MiningBenchMod in game (note from 2026-11-02, mining loop ran through MSCore).
- Exports marked **Not verified in game** were built and checked offline only.

## Error convention

Most functions that take `wchar_t* err, int errlen` follow the same rule:

- Return value `1` = success, `0` = failure.
- `err` gets the reason as a null-terminated wide string. `errlen` is the size of the buffer in `wchar_t` units. Longer text is truncated.
- Exceptions inside MSCore are caught and reported as `"exception"` with return `0`.
- Pass a buffer of at least 256 `wchar_t` to get useful messages.

Functions with a `why` buffer (`MSCore_find_item`, `MSCore_find_dock`) use the same buffer rule for the reason.

Functions that take a `void* obj` check that the object is alive before they use it. A dead object returns `0`, `NULL` or `-1`, depending on the function.

## Exports

All exports use the C calling convention and are declared `extern "C"`. Pointers are game `UObject*` values as `void*`. Chunk and location arrays hold 3 `long long` values in `X, Y, Z` order.

### Version

```c
int MSCore_api(void);   // returns 2
```

### Ship commands (in game, MiningBench)

```c
int MSCore_add_input(int receiver, void* cmd, wchar_t* err, int errlen);
// receiver = ship id. cmd = live input command UObject. Sends DMSPlayerController.AddInputCommand.
// Note: unlike the Lua cmd.send, this export does not apply the ShipMovementInputCommand type id fix.

int MSCore_view_send(int cid, long long yaw_raw, long long pitch_raw, wchar_t* err, int errlen);
// cid = PilotConsole ID. yaw_raw and pitch_raw are RAW values (65536 per degree). Lua steer.aim takes degrees.

int MSCore_throttle(int sid, int delta, wchar_t* err, int errlen);
// sid = ship id. Sends one ShipMovementInputCommand with DeltaThrottle = delta.
```

### Lookups

```c
void* MSCore_find_item(const char* res, wchar_t* why, int len);
// res = object name, display name (case-insensitive) or numeric ID as text. Returns the Item UObject,
// or NULL with `why` set on zero or several matches. Not verified in game.

int MSCore_find_dock(const char* name, wchar_t* why, int len);
// name = in-game dock name (DisplayName). Returns the dock id (> 0), -1 = none, -2 = several matches (`why` says which).
// Not verified in game.
```

### Orders (not verified in game)

```c
int MSCore_new_order(int receiver, const wchar_t* cls, wchar_t* err, int errlen);
// cls = class name without the path, for example a DeliverOrder class name. Receiver = ship id or parent order id.

int MSCore_update_order(int oid, int target, void* item, int cmp, int amount, wchar_t* err, int errlen);
// item = Item UObject or NULL. cmp and amount are integers. target = dock or 0.

int MSCore_delete_order(int sid, int oid, wchar_t* err, int errlen);

int MSCore_autopilot(int sid, int on, wchar_t* err, int errlen);
// on: 0 = off, nonzero = on.

int MSCore_skip_to(int sid, int idx, int* before, int* after, wchar_t* err, int errlen);
// before and after receive CurrentOrderIndex around the call. Either pointer may be NULL.

int MSCore_filter_send(int sid, int idx, void* item, int amount, int action, int remove, wchar_t* err, int errlen);
// action: 0 = LIMIT, 1 = BUFFER. remove != 0 removes row idx. item may be NULL (empty row).

int MSCore_filters_read(void* ship, MsFilterRow* out, int max);
// Returns the number of logistic filter rows, or -1 on failure. Copies min(count, max) rows into out.
// out = NULL or max = 0: count only.
```

### Warp and sectors (not verified in game)

```c
int MSCore_warp_send(int sid, int sys, const long long* ch3, const long long* arr3, int wrap, int set_target, wchar_t* err, int errlen);
// ch3 = chunk {X,Y,Z}. arr3 = raw arrival location {X,Y,Z}. wrap != 0 wraps in piloting on/off.
// set_target != 0 sets bSetTarget on the command.

int MSCore_arrival(void* ship, int sys, const long long* ch3, long long dist, long long* arr3, wchar_t* err, int errlen);
// Jump arrival point in sector ch3 of system sys. dist = warp arrival distance (raw). arr3 receives the raw location.

int MSCore_jump_cost(void* ship, int sys, const long long* ch3, int* cost, wchar_t* err, int errlen);
// cost receives the jump cost integer.

int MSCore_chunk_of(void* obj, long long* ch3);
// Returns 1 and fills ch3 with the chunk of obj, or 0.

int MSCore_axes_of(void* obj, const wchar_t* prop, long long* out3);
// Returns 1 and fills out3 with the X, Y, Z raw values of the struct property `prop` of obj, or 0.

int MSCore_sys_id(void* obj);
// Solar system id of obj, or 0 when unreadable.

int MSCore_sec_warp_dist(void* sec, long long* out);
// Returns 1 and writes GenParams.WarpArrivalDistance (raw) of the sector to *out, or 0.

int MSCore_sectors(MsSector* out, int max);
// Writes every SolarSystemSector into out[0..max). Returns the total found, or -1 on exception.
// out = NULL or max = 0: count only.
```

### Map pin (not verified in game)

```c
int MSCore_pin_ensure(void);
// Arms the map pin hook. Returns 1 when armed. Call every tick until it returns 1 (the controller
// does not exist when the mod loads).

int MSCore_pinned(void);
// Last map-pinned sector id, 0 = none.
```

### Not for mods

`start_mod` and `uninstall_mod` are the UE4SS entry points of the DLL. Do not call them.

## Structs

Both structs use the default C packing of a 64-bit build. Declare them exactly like this in your code. The sizes are checked with `static_assert` in MSCore.

### MsFilterRow (16 bytes)

| Offset | Field | Type |
|---|---|---|
| 0 | `item` | `void*` (Item UObject) |
| 8 | `amount` | `int32_t` |
| 12 | `action` | `int32_t` (0 LIMIT, 1 BUFFER) |

```cpp
struct MsFilterRow { void* item; int32_t amount; int32_t action; };
```

### MsSector (584 bytes)

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | 4 | `id` | save id of the sector |
| 4 | 4 | `sys` | solar system id |
| 8 | 4 | `nores` | number of ore entries in `ores` |
| 12 | 4 | `pad` | unused |
| 16 | 24 | `ch[3]` | chunk X, Y, Z (`long long`) |
| 40 | 24 | `he[3]` | half extents X, Y, Z (`long long`) |
| 64 | 8 | `warp_dist` | warp arrival distance (raw) |
| 72 | 320 | `ores[40]` | ore item UObject pointers, max 40 |
| 392 | 192 | `name[96]` | display name, `wchar_t`, null-terminated |

```cpp
struct MsSector {
    int32_t id;
    int32_t sys;
    int32_t nores;
    int32_t pad;
    long long ch[3];
    long long he[3];
    long long warp_dist;
    void* ores[40];
    wchar_t name[96];
};
```

The header `cpp\MSCore\msc_api.hpp` in the MSCore source has the same definitions. Copy the layout, not the file.

## Loading from another native mod

Load the MSCore DLL by path, then resolve the exports you need. MiningBenchMod uses a path relative to its own DLL folder.

```cpp
#include <windows.h>

typedef int (*MSCore_api_fn)(void);
typedef int (*MSCore_view_send_fn)(int, long long, long long, wchar_t*, int);

// dll_path: ue4ss\Mods\MSCore\dlls\main.dll, found from your own DLL folder
HMODULE h = LoadLibraryW(dll_path);
if (!h) return false;

auto api = reinterpret_cast<MSCore_api_fn>(GetProcAddress(h, "MSCore_api"));
if (!api || api() < 2) return false;   // API too old or missing

auto view_send = reinterpret_cast<MSCore_view_send_fn>(GetProcAddress(h, "MSCore_view_send"));
wchar_t err[256] = {0};
if (view_send && !view_send(cid, 0, 0, err, 256)) {
    // err holds the reason
}
```

`cid` is the pilot console id (`ID` property) of the ship's `PilotConsoleObject`.

Status: **in game (MiningBench)** for `MSCore_add_input`, `MSCore_view_send`, `MSCore_throttle`. Everything else **Not verified in game**.
