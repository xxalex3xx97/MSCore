---
title: Troubleshooting
parent: Players
nav_order: 3
---

# Troubleshooting

## Where the logs are

| Log | Where | What it holds |
|---|---|---|
| Manager log | `%TEMP%\MSCoreManager.log` (usually `C:\Users\<you>\AppData\Local\Temp\MSCoreManager.log`) | Every manager action: `install`, `add`, `remove`, errors. Lines are added at the end. |
| UE4SS log | The `ue4ss` folder in the game (`Windows\MainSequence\Binaries\Win64\ue4ss`) | Output of UE4SS and every mod, including `[MSCore]` lines. The file name is `UE4SS.log` by UE4SS default. **Not verified in game** for this install. |
| Selfcheck log | `ue4ss\Mods\MSCore\selfcheck.log` | Result of the MSCore self check. Written about 8 seconds after a world loads. |

## Reading selfcheck.log

Each line has one status:

- `OK`: the check passed.
- `BROKEN`: the check failed or raised an error. Send this line when you ask for help.
- `SKIP`: the check needs something that is not loaded yet (for example no world).
- `CHANGED`: the game exe size is not the one MSCore was verified with. The game was probably updated. MSCore may not work until it is updated.

The last line is a summary: `SUMMARY ok=... broken=... changed=... skip=...`.

## Stale cache

The manager asks GitHub for `manifest.txt` with a `?t=<time>` query string. That makes GitHub return the current file and not an old cached copy.

If the manager shows an old version after a new release:

1. Click **Check** again.
2. Make sure the new release is marked **Latest** on GitHub. The manager reads `releases/latest`.
3. Before version 0.5.1, the query string could be dropped and an old manifest could be read. Update the manager to 0.5.1 or later.

## Game running guard

If the manager says "Game is running - close it first", close Main Sequence Demo completely, check that no `MainSequence` process is left in Task Manager, and try again.

## The manager does not find the game

Click **Browse game folder** and pick the `Main Sequence Demo` folder. The folder must contain `Windows\MainSequence\Binaries\Win64`.

## The mod does not load

1. Check that `ue4ss\Mods\mods.txt` has `MSCore : 1` and a line for your mod (`<ModName> : 1`).
2. Check the UE4SS log for errors from your mod.
3. Check that the mod folder has an `enabled.txt` file. If it does not, add the line to `mods.txt` yourself.

## Some MSCore features do nothing

The native DLL must be in `ue4ss\Mods\MSCore\dlls\main.dll`. Without it, Lua calls that need the native part return the reason "native DLL not loaded". The selfcheck log shows this as `BROKEN`.

## Mod update does not apply

Updates are staged and applied when the game starts. Close the game fully, start it again, and check the version in the manager. The manager log shows staging lines.
