# Windows Terminal

Windows Terminal provides a modern, tabbed terminal for PowerShell, Command Prompt, and WSL. Install it to make commandline work smoother for all trainings.

## Install via Microsoft Store (recommended)

- Open the Microsoft Store, search for "Windows Terminal," and install.
- This keeps updates automatic and avoids manual downloads.

## Install via Winget

```powershell
winget install --id Microsoft.WindowsTerminal -e
```

- If prompted about source agreements, accept to continue.

## Verify

```powershell
wt -v
```

- Launch Windows Terminal from the Start menu; ensure you see tabs for PowerShell and Command Prompt.

## Configure Basics

- Set default profile to PowerShell (or WSL if you use it most).
- Enable copy-on-select and paste with Ctrl+V (Settings > Interaction).
- Optional: adjust font to Cascadia Code and set a readable theme.
