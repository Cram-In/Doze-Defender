# Doze Defender

> A tiny, dependency-free Windows utility that keeps your PC and display awake — and puts everything back to normal the moment you close it.

![Platform](https://img.shields.io/badge/platform-Windows-0078D6?logo=windows)
![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-5391FE?logo=powershell&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

Doze Defender (the **Sleeper** window) is a single PowerShell script with a small GUI. While it is open, Windows will not go to sleep and the screen will not turn off. Close the window and your normal power settings apply again — no registry edits, no power-plan changes, nothing left behind.

---

## Features

- **Keeps the system and display awake** using the official Windows API (`SetThreadExecutionState`).
- **No side effects** – does not modify power plans, registry, or group policy settings.
- **Automatic cleanup** – normal sleep behaviour is restored when the window closes, even after an error.
- **Clean, minimal UI** – status indicator and a live running-time counter.
- **No console window** – the launcher starts the script silently in the background.
- **No installation, no admin rights, no dependencies** – just Windows and the built-in PowerShell.
- **Lightweight logging** to `%TEMP%\keepawake.log`.

## Requirements

| Requirement | Notes |
|-------------|-------|
| Windows 10 (1809+) or Windows 11 | `conhost.exe --headless` is used by the launcher |
| Windows PowerShell 5.1 | Included with Windows; PowerShell 7 also works when run directly |
| .NET Framework | Used for Windows Forms; included with Windows |

## Getting started

### 1. Download

```powershell
git clone https://github.com/Cram-In/Doze-Defender.git
cd Doze-Defender
```

Or download the ZIP from GitHub and extract it.

> **Tip:** If you downloaded the ZIP, Windows may mark the files as coming from the internet. Unblock them once with:
>
> ```powershell
> Get-ChildItem -Path . | Unblock-File
> ```

### 2. Run

**Option A – double-click (recommended)**

Double-click **`sleeper.cmd`**. The Sleeper window appears without any console window.

**Option B – from PowerShell**

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\sleeper.ps1
```

### 3. Stop

Click **Stop and close** (or simply close the window). Your regular sleep and screen-off settings are restored immediately.

## How it works

Windows lets applications tell the power manager that they are busy. Doze Defender calls the Win32 function [`SetThreadExecutionState`](https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-setthreadexecutionstate) with:

| Flag | Purpose |
|------|---------|
| `ES_CONTINUOUS` | Keep the state in effect until it is explicitly changed |
| `ES_SYSTEM_REQUIRED` | Prevent the system from going to sleep |
| `ES_DISPLAY_REQUIRED` | Prevent the display from turning off |

When the window closes, the script calls `SetThreadExecutionState(ES_CONTINUOUS)` to clear the request. Because the request is tied to the running process, Windows also releases it automatically if the process is terminated unexpectedly.

This is the same mechanism used by media players and presentation software — it does **not** simulate mouse movement or key presses.

### Verify it is working

While Doze Defender is running, open an elevated terminal and run:

```powershell
powercfg /requests
```

You should see a `powershell.exe` entry under both **DISPLAY** and **SYSTEM**.

## Project structure

```
Doze-Defender/
├── sleeper.cmd   # Silent launcher (no console window)
├── sleeper.ps1   # Main script: GUI + keep-awake logic
├── LICENSE
└── README.md
```

## Optional: start with Windows

1. Press `Win + R`, type `shell:startup`, and press **Enter**.
2. Create a shortcut to `sleeper.cmd` in that folder.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Nothing happens when double-clicking `sleeper.cmd` | Run `sleeper.ps1` directly from PowerShell (Option B) to see any error messages. |
| *"running scripts is disabled on this system"* | Use the launcher or the `-ExecutionPolicy Bypass` command above, or run `Unblock-File` on the downloaded files. |
| Red status dot / *"Could not enable keep-awake mode"* | Check `%TEMP%\keepawake.log` for details. |
| PC still locks the screen | The tool prevents **sleep** and **display-off**; a lock enforced by organisational policy (screen saver / inactivity lock) may still apply. |
| Laptop sleeps when the lid is closed | Lid-close actions are handled by your power settings and are not overridden. |

## Logging

Start, stop, and error events are appended to:

```
%TEMP%\keepawake.log
```

Example:

```
2026-10-15 09:12:03 - Sleeper started
2026-10-15 17:45:51 - Sleeper stopped
```

## Responsible use

Doze Defender is intended to prevent interruptions during long-running tasks such as downloads, builds, presentations, or remote sessions. Please respect your organisation's IT and security policies when using it on managed devices.

## Contributing

Issues and pull requests are welcome. For larger changes, please open an issue first to discuss what you would like to change.

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-change`
3. Commit your changes: `git commit -m "Add my change"`
4. Push the branch: `git push origin feature/my-change`
5. Open a pull request

## License

This project is licensed under the [MIT License](LICENSE).
