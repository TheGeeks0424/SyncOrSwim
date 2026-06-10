# SyncOrSwim

Co-op multiplayer sync fixes for **[Nitrox](https://github.com/SubnauticaNitrox/Nitrox) 1.8.1.0** (Subnautica build 1.22.83031).

> Status: **pre-release.** Patches build, deploy, and run; end-to-end soak testing with two players is ongoing. Issues welcome.

## What this fixes

| # | Symptom (before) | Behaviour (after) |
|---|---|---|
| 1 | Items deposited by one player into a shared container (lockers, wall lockers, Prawn/Seamoth storage, …) did not appear for the other player live. Deposits could be lost entirely on rejoin because the server never persisted them. | Deposits are visible immediately to all players who can see the container, are persisted server-side, and survive rejoin. Idempotent on the item's unique ID — no duplication. |
| 2 | Hull-breach flood water level did not propagate between players. Repair/drain only converged after a rejoin. | The simulating client broadcasts water level + hull strength to peers at 1 Hz. Non-owners see the same flood live, and rejoining players are rehydrated mid-flood instead of starting dry. |

**Who needs to install it:** everyone — both players **and the server host** run the same installer.

- Fix 1 (storage) is carried entirely by the client DLLs and works even against an unpatched server, but both *players* need it for deposits to sync in both directions.
- Fix 2 (flooding) introduces one new metadata payload (`BaseFloodMetadata`, carried inside Nitrox's existing `EntityMetadataUpdate` packet), so the **host** must also run the installer — it swaps in the patched model DLL the server loads. The protocol version is unchanged and unpatched players can still connect and play; they just don't get the fixes.

## Install (binary distribution)

The simplest path: download `SyncOrSwim-Installer.zip` from [Releases](../../releases), extract anywhere (keep the `server` subfolder), and double-click `Install-SyncOrSwim.bat`. Run it on every machine involved — both players and the host. Full instructions in [INSTALL.md](INSTALL.md).

## Source

The complete modified Nitrox tree is public at [TheGeeks0424/Nitrox, branch `fix/storage-and-flood-sync`](https://github.com/TheGeeks0424/Nitrox/tree/fix/storage-and-flood-sync) — three commits on top of the upstream `1.8.1.0` tag. A changed-files overlay is also attached to each release as `syncorswim-source-<tag>.zip`.

### Build from source

1. `git clone https://github.com/TheGeeks0424/Nitrox.git`
2. `git -C Nitrox checkout fix/storage-and-flood-sync`
3. Install **.NET 9 SDK** (the `Nitrox.Analyzers` source generator requires Roslyn 4.11+).
4. Set `SUBNAUTICA_INSTALLATION_PATH` to your Subnautica install dir (the build references `Assembly-CSharp.dll`).
5. `dotnet build Nitrox/NitroxPatcher/NitroxPatcher.csproj -c Release -f net472`
6. Client DLLs land in each project's `bin/Release/net472/` — copy `NitroxClient.dll`, `NitroxPatcher.dll`, `Nitrox.Model.Subnautica.dll` into your Nitrox launcher's `lib/net472/`. For the host, additionally copy `Nitrox.Model.Subnautica/bin/Release/net9.0/Nitrox.Model.Subnautica.dll` into `lib/`.

## What changed

Seven files on top of upstream Nitrox 1.8.1.0 (2 modified, 5 new):

| File | Change | Purpose |
|---|---|---|
| `NitroxClient/GameLogic/ItemContainers.cs` | **modified** | Routes all container deposits through `Items.MovedIntoInventory` (full `EntitySpawnedByClient` payload, idempotent at every hop) instead of the thin `EntityReparented` packet that was silently dropped when the moved entity was unknown to the server or the receiving client. Closes fix 1. |
| `Nitrox.Model.Subnautica/.../Metadata/EntityMetadata.cs` | **modified** | Registers `BaseFloodMetadata` with the protobuf serializer (`ProtoInclude 85`) so flood sync works under both Nitrox serializer modes. |
| `Nitrox.Model.Subnautica/.../Metadata/BaseFloodMetadata.cs` | **new** | DTO carrying per-cell water level + base hull strength. |
| `NitroxClient/GameLogic/Spawning/Metadata/BaseFloodSyncState.cs` | **new** | Per-base throttle/last-sent bookkeeping for the flood broadcast. |
| `NitroxClient/.../Extractor/BaseFloodMetadataExtractor.cs` | **new** | Reads `BaseFloodSim.cellWaterLevel` + `BaseHullStrength.totalStrength`. |
| `NitroxClient/.../Processor/BaseFloodMetadataProcessor.cs` | **new** | Applies received state under `PacketSuppressor<EntityMetadataUpdate>` to avoid feedback loops. Length-checks the cell array against the local base shape. |
| `NitroxPatcher/Patches/Dynamic/BaseFloodSim_Update_Patch.cs` | **new** | Harmony postfix on `BaseFloodSim.Update()`. Sim-owner check via `SimulationOwnership.HasAnyLockType`. Throttled broadcast at 1 Hz. |

## Tuning knobs

The flood-sync throttle is at the top of `BaseFloodSim_Update_Patch.cs`:

```csharp
private const float THROTTLE_SECONDS = 1.0f;
```

If you see water-level "stutter" or visible step-wise updates, drop this to `0.3f` (more bandwidth, smoother). If you see bandwidth concerns with many active leaks, raise it to `2.0f` (less smooth, lighter on the wire).

## Authors

- **Connor** ([@TheGeeks0424](https://github.com/TheGeeks0424))
- **Zach**

Upstream Nitrox is the work of the [SubnauticaNitrox contributors](https://github.com/SubnauticaNitrox/Nitrox/graphs/contributors). This project is a derivative work, intended to be offered upstream once play-tested.

## License

GPL-3.0 — see [LICENSE](LICENSE). This is required because Nitrox itself is GPL-3.0; any redistribution of modified Nitrox binaries inherits that license. See [NOTICE.md](NOTICE.md) for upstream credit and the exact commit this is derived from.
