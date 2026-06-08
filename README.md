# SyncOrSwim

Co-op multiplayer sync fixes for **[Nitrox](https://github.com/SubnauticaNitrox/Nitrox) 1.8.1.0** (Subnautica build 1.22.83031).

> Status: **public test release.** Patches build, deploy, and run; end-to-end soak testing with two players is ongoing. Issues welcome.

## What this fixes

| # | Symptom (before) | Behaviour (after) |
|---|---|---|
| 1 | Items deposited by one player into a shared container (Prawn storage, vehicle storage, wall locker) did not appear for the other player live. Deposits could be lost on rejoin. | Deposits are visible immediately to all players who can see the container, and persist across rejoin. |
| 2 | Hull-breach flood water level did not propagate between players. Repair/drain only converged after a rejoin. | The simulating client broadcasts water level + hull strength to peers at 1 Hz. Non-owners and rejoining players see the same flood state. |

Both fixes are **client-side only**. The server (host) does not need any change. No wire-protocol changes. No new packet types.

## Install (binary distribution)

The simplest path: download `SyncOrSwim-Installer.zip` from [Releases](../../releases), extract anywhere, and double-click `Install-SyncOrSwim.bat`. Full instructions in [INSTALL.md](INSTALL.md).

**Both players in a co-op session must install this for the fix to fully work.**

## Build from source

The full corresponding source for the binaries is attached to each release as `syncorswim-source-<tag>.zip`. To rebuild from scratch:

1. Download `syncorswim-source-<tag>.zip` from the matching [Release](../../releases) and extract it.
2. `git clone https://github.com/SubnauticaNitrox/Nitrox.git`
3. `git -C Nitrox checkout tags/1.8.1.0`
4. Copy the contents of the extracted `src/` over the matching paths in the clone (1 file edited, 4 files new).
5. Install **.NET 9 SDK** (the `Nitrox.Analyzers` source generator requires Roslyn 4.11+).
6. Set `SUBNAUTICA_INSTALLATION_PATH` to your Subnautica install dir (the build references `Assembly-CSharp.dll`).
7. `dotnet build Nitrox/NitroxPatcher/NitroxPatcher.csproj -c Release -f net472`
8. Resulting DLLs land in each project's `bin/Release/net472/` — copy them into your Nitrox launcher's `lib/net472/`.

## What changed

The source zip contains five files this project contributes on top of upstream Nitrox 1.8.1.0:

| File | Change | Purpose |
|---|---|---|
| `NitroxClient/GameLogic/ItemContainers.cs` | **modified** | Routes all container deposits through `Items.MovedIntoInventory` (full `EntitySpawnedByClient` payload) so receiving clients can always lazy-create the entity locally. Closes bug 1 + 2. |
| `Nitrox.Model.Subnautica/DataStructures/GameLogic/Entities/Metadata/BaseFloodMetadata.cs` | **new** | DTO carrying per-cell water level + base hull strength. |
| `NitroxClient/GameLogic/Spawning/Metadata/Extractor/BaseFloodMetadataExtractor.cs` | **new** | Reads `BaseFloodSim.cellWaterLevel` + `BaseHullStrength.totalStrength`. |
| `NitroxClient/GameLogic/Spawning/Metadata/Processor/BaseFloodMetadataProcessor.cs` | **new** | Applies received state under `PacketSuppressor<EntityMetadataUpdate>` to avoid feedback loops. Length-checks the cell array against the local base shape. |
| `NitroxPatcher/Patches/Dynamic/BaseFloodSim_Update_Patch.cs` | **new** | Harmony postfix on `BaseFloodSim.Update()`. Sim-owner check via `SimulationOwnership.HasAnyLockType`. Throttled broadcast at 1 Hz via `Entities.BroadcastMetadataUpdateThrottled`. |

## Tuning knobs

The flood-sync throttle is at the top of `BaseFloodSim_Update_Patch.cs`:

```csharp
private const float THROTTLE_SECONDS = 1.0f;
```

If you see water-level "stutter" or visible step-wise updates, drop this to `0.3f` (more bandwidth, smoother). If you see bandwidth concerns with many active leaks, raise it to `2.0f` (less smooth, lighter on the wire).

## Authors

- **Connor Sorey** ([@TheGeeks0424](https://github.com/TheGeeks0424))
- **Zach**

Upstream Nitrox is the work of the [SubnauticaNitrox contributors](https://github.com/SubnauticaNitrox/Nitrox/graphs/contributors). This project is a derivative work.

## License

GPL-3.0 — see [LICENSE](LICENSE). This is required because Nitrox itself is GPL-3.0; any redistribution of modified Nitrox binaries inherits that license. See [NOTICE.md](NOTICE.md) for upstream credit and the exact commit this is derived from.
