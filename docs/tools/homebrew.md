# Homebrew (macOS)

Homebrew is a package manager that makes it easy to install command line tools on macOS.

- Consistent installs for CLI tools across machines.
- Easy updates: `brew update && brew upgrade`.
- Dependency management: `brew info <package>` shows versions and requirements.

## Prerequisites

Ensure Command Line Tools are installed:

```bash
xcode-select --install
```

## Installation

- Run the official installer:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Add Homebrew to your shell path (follow the installer's final hints)

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

- Verify:

```bash
brew --version
brew doctor
```
