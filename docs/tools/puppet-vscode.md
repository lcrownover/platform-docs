# Puppet VS Code Extension

The [Puppet extension](https://marketplace.visualstudio.com/items?itemName=puppet.puppet-vscode) for Visual Studio Code adds language support for Puppet manifests. It gives you syntax highlighting, autocompletion, hover documentation, and linting as you type.

!!! note
    Without PDK installed, the extension still provides basic syntax highlighting. Features like autocompletion, hover documentation, and full validation require [PDK](pdk.md).

## Prerequisites

| Tool | Why You Need It |
|------|-----------------|
| [Visual Studio Code](vscode.md) | The extension runs inside VS Code |

## Installation

1. Open VS Code
2. Go to Extensions (`Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on macOS)
3. Search for "Puppet" and install the one published by **Puppet**

## What You Get

- **Syntax highlighting** for `.pp`, `.epp`, and Puppetfile files
- **Autocompletion** for resource types, parameters, and built-in functions
- **Hover info** for resource types and parameters
- **Linting** powered by puppet-lint (if available)

## PDK Not Installed

When the extension starts, it looks for the [Puppet Development Kit (PDK)](https://www.puppet.com/docs/pdk/latest/pdk.html) on your system. If PDK isn't installed, you'll see an error in the VS Code output panel about not being able to find it.

!!! note
    You can safely ignore/dismiss this error. The extension still provides syntax highlighting, basic autocompletion, and code formatting without PDK. The features that depend on PDK (like full validation and some advanced IntelliSense) won't work, but for editing team modules and Hiera data, the basic functionality is all you need.
