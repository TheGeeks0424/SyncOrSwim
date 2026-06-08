# NOTICE

This project is a derivative work of **Nitrox**, the multiplayer mod for Subnautica.

## Upstream

- Upstream project: <https://github.com/SubnauticaNitrox/Nitrox>
- License: GNU General Public License, version 3 (GPL-3.0)
- Commit this work is derived from: tag `1.8.1.0` (commit `aec1f6e4`)
- Direct link: <https://github.com/SubnauticaNitrox/Nitrox/tree/1.8.1.0>

The full corresponding source code for the binaries distributed in our releases is attached to each release as `syncorswim-source-<tag>.zip`. That zip contains the modified files this project contributes on top of upstream Nitrox 1.8.1.0; apply them over a fresh upstream checkout at the tag above to reproduce the binaries. See [README.md](README.md#build-from-source) for step-by-step build instructions. This satisfies GPL-3.0 §6 (Conveying Non-Source Forms) by providing the source code through publicly available means alongside the binary.

## What this project adds

- One edited file: `NitroxClient/GameLogic/ItemContainers.cs`
- Four new files:
  - `Nitrox.Model.Subnautica/DataStructures/GameLogic/Entities/Metadata/BaseFloodMetadata.cs`
  - `NitroxClient/GameLogic/Spawning/Metadata/Extractor/BaseFloodMetadataExtractor.cs`
  - `NitroxClient/GameLogic/Spawning/Metadata/Processor/BaseFloodMetadataProcessor.cs`
  - `NitroxPatcher/Patches/Dynamic/BaseFloodSim_Update_Patch.cs`

All other files in the released DLLs are unmodified upstream Nitrox 1.8.1.0 code, built with the upstream's own build system.

## Trademarks

"Subnautica" and related marks are the property of Unknown Worlds Entertainment. "Nitrox" is the name used by the upstream project. This project ("SyncOrSwim") is not affiliated with, endorsed by, or sponsored by Unknown Worlds Entertainment or the upstream Nitrox maintainers.
