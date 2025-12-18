# Git

Git is a distributed version control system that tracks changes to files over time, allowing you to save checkpoints (commits), work in parallel (branches), and safely combine work (merges).
It makes collaboration and change management reliable:

- Provides complete history for auditing and rollback
- Facilitates code review with content diffs
- Enables a low-risk way to experiment without breaking the main codebase

## Installation Instructions

### Windows

Manual installation using [binaries from the official site](https://git-scm.com/install/windows)

The default settings in the installer should work just fine.

or

Install using WinGet:

```powershell
winget install --id Git.Git -e --source winget
```

### macOS

- Command Line Tools (built-in, but less up to date):

```bash
xcode-select --install
```

- Homebrew (preferred for current releases):

```bash
brew install git
```

### Linux (Debian/Ubuntu)

- Install from apt:

  ```bash
  sudo apt update
  sudo apt install -y git
  ```

## Configure Git for your machine

There are some settings you should configure prior to using it. You only need to set these once!

```bash
git config --global user.name "Your Name"          # the name that shows up in your commit history
git config --global user.email "you@uoregon.edu"   # your email
git config --global pull.rebase true               # prefer rebase over merge when pulling
```
