# puppet-editor-services

puppet-editor-services is the Puppet Language Server. It provides real-time syntax checking, autocompletion, hover documentation, and code navigation for Puppet files in editors that support the Language Server Protocol (LSP). If you're writing Puppet code in [VS Code](vscode.md), this is what powers the Puppet extension behind the scenes.

## Prerequisites

| Tool | Why You Need It |
|------|-----------------|
| [Ruby](ruby.md) | puppet-editor-services is a Ruby gem |

## Installation

```bash
gem install puppet-editor-services
```

## Verify Installation

```bash
puppet-languageserver --version
```

If you get "command not found," your gem bin directory isn't in your `PATH`. See the [Ruby tool guide](ruby.md#verify-gems-are-in-path) for troubleshooting.

## VS Code Setup

The [Puppet extension](https://marketplace.visualstudio.com/items?itemName=puppet.puppet-vscode) for VS Code uses puppet-editor-services automatically. Install the extension:

1. Open VS Code
2. Go to Extensions (`Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on macOS)
3. Search for "Puppet" and install the one published by **Puppet**

Once installed, open any `.pp` file and the language server will start. You'll get:

- **Syntax validation** -- errors underlined as you type
- **Autocompletion** -- resource types, parameters, and variables
- **Hover info** -- documentation when you hover over resource types and functions
- **Formatting** -- auto-format on save (if enabled in VS Code settings)

!!! tip
    If the extension can't find the language server, open VS Code settings and set the `puppet.installDirectory` or ensure `puppet-languageserver` is in your `PATH`.

## Other Editors

Any editor with LSP support can use puppet-editor-services. You'll need to configure it to run `puppet-languageserver --stdio` as the language server command. Consult your editor's LSP documentation for details.
