# Git

Git is a version control system that tracks changes to files over time. You save checkpoints (commits), work on isolated changes (branches), and combine work together (merges). It's the foundation for modern infrastructure-as-code, config management, and collaborative workflows.

## Installation

### macOS

**Homebrew** (recommended)

```bash
brew install git
```

### Windows

Download the installer from [git-scm.com](https://git-scm.com/download/win) and run it. There are many options during installation -- accept all the defaults. You will need to open a new terminal window after the install is complete.

!!! tip
    If you're using [WSL](wsl.md), you'll also want to install Git inside WSL separately using the Linux instructions below.

### Linux (Ubuntu) / WSL

```bash
sudo apt update && sudo apt install -y git
```

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

Use your real name and work email. When someone runs `git log` or `git blame`, your name/email is what they'll see.

**Recommended settings:**

```bash
git config --global pull.rebase true    # cleaner history when pulling changes
```

Verify your config:

```bash
git config --global --list
```
