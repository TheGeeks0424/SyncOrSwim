# Installing SyncOrSwim

Installation has two parts. Skip part A if you already have Nitrox 1.8.1.0 running on your system today.

You only need **.NET 9 Desktop Runtime** — not the SDK. Runtime ≈ 60 MB, SDK ≈ 200 MB.

---

## Part A — Install Nitrox 1.8.1.0 (skip if already running)

### 1. Install .NET 9 Desktop Runtime

**Easiest (PowerShell, winget):**
```powershell
winget install Microsoft.DotNet.DesktopRuntime.9
```

**Manual:** download from <https://dotnet.microsoft.com/download/dotnet/9.0/runtime> → pick **".NET Desktop Runtime 9.0"** → **Windows x64** → run installer.

Verify in a new PowerShell window:
```powershell
dotnet --list-runtimes
```
You should see `Microsoft.WindowsDesktop.App 9.0.x`.

### 2. Install Subnautica

If not already installed, install via Steam (or Epic). Launch it once and let it finish updating — Nitrox needs Subnautica build **1.22.83031**.

### 3. Install Nitrox 1.8.1.0

1. Download `Nitrox_1.8.1.0_win_x64.zip` from <https://github.com/SubnauticaNitrox/Nitrox/releases/tag/1.8.1.0> (under "Assets").
2. Extract to a fixed folder, e.g. `C:\Nitrox_1.8.1.0_win_x64\`. Avoid `Program Files` — Nitrox needs write access for save files.
3. Double-click `Nitrox.Launcher.exe`. Let it discover Subnautica (or point it at the install path). Close the launcher.

---

## Part B — Install SyncOrSwim on top

### 1. Get the installer

Download `SyncOrSwim-Installer.zip` from the [latest release](../../releases).

### 2. Unblock the zip

Windows blocks scripts from "internet" zip files by default. Right-click the downloaded zip → **Properties** → tick **Unblock** if it's there → **OK**.

### 3. Extract and run

1. Extract the zip to any folder (Desktop is fine). Everything must end up in the **same folder**, including the `server` subfolder (it holds the DLL the Nitrox *server* loads — the installer needs it even on machines that only play).
2. Make sure Nitrox Launcher, Subnautica, **and the Nitrox server** are **closed**.
3. Double-click **`Install-SyncOrSwim.bat`**.
4. If SmartScreen appears: **More info** → **Run anyway**.
5. The script auto-detects your Nitrox folder. If it can't, paste the full path to the folder containing `Nitrox.Launcher.exe` when prompted.
6. Wait for `Done.` Press Enter to close.

### 4. Verify

Launch `Nitrox.Launcher.exe`. It should start as normal — there's no version label change to look at; the only visible effect is in-game behaviour during a multiplayer session.

---

## Uninstall

Double-click `Uninstall-SyncOrSwim.bat`. It restores the most recent backup of each patched DLL.

---

## Everyone installs this — both players AND the host

Run the same installer on every machine involved:

- **Players:** a mixed session (one player patched, one stock) only **partially** works — the patched player's deposits sync; the unpatched player's deposits still desync.
- **Host:** the machine running the Nitrox server must also run the installer, or base flooding will not sync (the server needs the patched model DLL to understand the new flood-sync message). The storage fix works even with an unpatched host.

Unpatched players can still connect and play normally either way.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Launcher won't open / crashes on launch | .NET 9 Desktop Runtime not installed (Part A, step 1). |
| Installer says "Patched DLLs missing" | The zip wasn't fully extracted — all files (including the `server` subfolder) must sit next to the .bat file. |
| Installer says "Close these processes first" | Quit Nitrox Launcher, Subnautica, and the Nitrox server fully, then re-run. |
| `.bat` opens and immediately closes with no output | Right-click `Install-SyncOrSwim.ps1` → **Properties** → **Unblock** → **OK**, then re-run the .bat. |
| Connected fine but deposits still desync | Confirm your partner also ran the installer. |
| Storage syncs but flooding doesn't | Confirm the **host** machine also ran the installer (flood sync requires the patched server DLL). |
| Water level still jumps after a breach | Expected if no player has been in the base while it was flooding — the simulating client broadcasts at 1 Hz, so the receiver catches up within ~1 second of being in range. |

If something behaves differently than the table above, open an issue with the output of the installer script + your `%APPDATA%\Nitrox\logs\` from the failing session.
