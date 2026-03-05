# Windows Subsystem for Linux (WSL)

WSL lets you run a full Linux environment directly on Windows—no virtual machine, no dual-boot. You get a real Linux shell, real Linux tools, and real Linux package managers, all integrated with your Windows filesystem.

## Why Use WSL for Development?

Most servers, containers, and CI/CD pipelines run Linux. When you develop on Windows but deploy to Linux, you're constantly fighting small differences: path separators, line endings, case sensitivity, shell syntax, missing tools.

WSL eliminates this friction:

- **Same tools as production.** Use the same shell, package manager, and utilities you'll find on your servers. No more "works on my machine" because your machine *is* Linux.
- **Native performance.** WSL 2 runs a real Linux kernel. File operations, compilation, and container workloads run at near-native speed.
- **Seamless integration.** Access Windows files from Linux (`/mnt/c/`), run Linux commands from PowerShell, and use VS Code's Remote-WSL extension to edit Linux files with full IDE support.
- **Better tooling support.** Many development tools (Docker, Kubernetes, Terraform, Ansible, Puppet) are Linux-first. Documentation, tutorials, and Stack Overflow answers assume Linux. Stop translating—just use Linux.

If you're doing infrastructure work, writing scripts, or working with containers, WSL is the path of least resistance on Windows.

## Installation

### Requirements

- Windows 10 version 2004+ (Build 19041+) or Windows 11
- Virtualization enabled in BIOS (usually on by default)

### Install WSL

Open PowerShell as Administrator and run:

```powershell
wsl --install --no-distribution
```

This installs the WSL 2 engine and kernel without a default distribution. Restart when prompted.

### Install a Distribution

After restarting, open PowerShell and check what distributions are available:

```powershell
wsl --list --online
```

You'll see output similar to:

```
NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Ubuntu-22.04                    Ubuntu 22.04 LTS
Ubuntu-24.04                    Ubuntu 24.04 LTS
...
```

Install the latest Ubuntu LTS:

```powershell
wsl --install -d Ubuntu-24.04
```

!!! tip
    Always choose the latest LTS (Long Term Support) release. LTS versions get 5 years of security updates and match what you're most likely to find on production Linux servers. If you see a newer LTS than 24.04 in the list, install that one instead.

Ubuntu will launch and prompt you to create a username and password. This is your Linux user—separate from your Windows account and doesn't need to match it.

### Verify Installation

Open a new PowerShell window:

```powershell
wsl --version
```

You should see WSL version 2.x.x and a kernel version.

Launch your Linux environment:

```powershell
wsl
```

You're now in a bash shell. Run `uname -a` to confirm you're in Linux.

## Basic Configuration

### Update Packages

First thing after install—update your package list:

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Common Tools

```bash
sudo apt install -y git curl wget unzip
```

### Access Windows Files

Your Windows drives are mounted under `/mnt/`:

```bash
cd /mnt/c/Users/YourName/Documents
ls
```

### Access Linux Files from Windows

Your Linux home directory is accessible from Windows Explorer:

```
\\wsl$\Ubuntu\home\yourusername
```

Or open Explorer from the Linux terminal:

```bash
explorer.exe .
```

## Using WSL with VS Code

Install the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension in VS Code.

Then from your Linux terminal, open any folder in VS Code:

```bash
code .
```

VS Code connects to WSL and runs extensions inside Linux. You get full IntelliSense, debugging, and terminal access—all running in your Linux environment.

## Tips

- **Keep projects in Linux filesystem.** Store code in `~/projects`, not `/mnt/c/`. File operations are significantly faster on the Linux filesystem.
- **Use Windows Terminal.** It handles WSL, PowerShell, and CMD in tabs with proper Unicode support. See [Windows Terminal](windows-terminal.md).
- **One distro is usually enough.** Ubuntu is the default and has the best support. You *can* install multiple distros, but you probably don't need to.
