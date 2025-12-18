# Prerequisites

Get these tools installed before starting. Verify each command so the lesson can move quickly for everyone.

## Required Tools
- Git CLI 2.39+ (for basic features and security fixes)
- Text editor (VS Code or similar)
- Terminal or PowerShell

## Install on macOS
1. Install Git (pick one):
   - Command Line Tools: `xcode-select --install`
   - Homebrew: `brew install git`
2. Install VS Code: `brew install --cask visual-studio-code` (or download from code.visualstudio.com).
3. Verify:
   ```bash
   git --version
   git config --global --list
   ```

## Install on Windows
1. Install Git for Windows:
   - Winget: `winget install --id Git.Git -e`
   - Chocolatey: `choco install git -y`
   - Or download from git-scm.com and choose “Git from the command prompt.”
2. Install VS Code:
   - Winget: `winget install --id Microsoft.VisualStudioCode -e`
   - Chocolatey: `choco install vscode -y`
3. Verify (PowerShell):
   ```powershell
   git --version
   git config --global --list
   ```

## Configure Git (run once)
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.ff only
```

## Files You’ll Create
- A practice repo in a scratch directory you can delete later.
- Small text files for commits and merge exercises.

If any install step fails, pause and resolve before class so the instructor-led session stays smooth for everyone.
