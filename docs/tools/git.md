# Git

Git is a version control system that tracks changes to files over time. You save checkpoints (commits), work on isolated changes (branches), and combine work together (merges). It's the foundation for modern infrastructure-as-code, config management, and collaborative workflows.

## Installation

### macOS

**Option 1: Xcode Command Line Tools** (quick, but may be outdated)

```bash
xcode-select --install
```

**Option 2: Homebrew** (recommended—stays current)

```bash
brew install git
```

### Linux / WSL

```bash
sudo apt update && sudo apt install -y git
```

!!! note "Windows users"
    Install Git inside [WSL](wsl.md), not on Windows itself. This keeps your tooling consistent with the Linux environments you'll deploy to.

## Verify Installation

```bash
git --version
```

You should see something like `git version 2.x.x`.

## Initial Configuration

Set these once per machine. They identify you in every commit you make.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@uoregon.edu"
```

Use your real name and work email—this is attribution, not authentication. When someone runs `git log` or `git blame`, your name is what they'll see.

**Recommended settings:**

```bash
git config --global init.defaultBranch main    # new repos start with 'main' instead of 'master'
git config --global pull.rebase true           # cleaner history when pulling changes
```

Verify your config:

```bash
git config --global --list
```
