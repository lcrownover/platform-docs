# Prerequisites

Get set up before starting. Use the shared tools guide for Git installs and confirm your editor works.

## Required Tools
- Git CLI 2.39+ (for basic features and security fixes)
- Text editor (VS Code or similar)
- Terminal or PowerShell

## Install Git
- Follow the shared instructions: [Install Git (Windows, macOS, Linux)](../tools/git.md).
- After installation, verify:
  ```bash
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

If any install step fails, pause and resolve before continuing so the workflow stays smooth.
