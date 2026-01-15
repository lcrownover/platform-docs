# Windows Terminal

Windows Terminal is a modern, tabbed terminal for Windows that handles PowerShell, Command Prompt, and [WSL](wsl.md) in one app. It supports Unicode, GPU-accelerated text rendering, and custom themes—everything the legacy console lacks.

## Installation

**Option 1: Microsoft Store** (recommended—auto-updates)

Open the Microsoft Store, search for "Windows Terminal," and install.

**Option 2: WinGet**

```powershell
winget install --id Microsoft.WindowsTerminal -e
```

## Verify Installation

Launch Windows Terminal from the Start menu. You should see a tabbed interface with PowerShell as the default.

Check the version:

```powershell
wt -v
```

## Set WSL as Default Profile

If you're using [WSL](wsl.md) for development, make it your default so new tabs open directly into Linux.

1. Open Windows Terminal
2. Press `Ctrl+,` to open Settings (or click the dropdown arrow → Settings)
3. Under **Startup**, find **Default profile**
4. Select **Ubuntu** (or your WSL distro) from the dropdown
5. Click **Save**

New tabs and windows now open in WSL by default. You can still open PowerShell or CMD tabs from the dropdown menu.

## Tips

- **Split panes:** `Alt+Shift+D` splits the current tab. Useful for running a command while watching logs.
- **Zoom:** `Ctrl+=` and `Ctrl+-` adjust font size on the fly.
