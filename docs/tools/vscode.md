# Visual Studio Code

Visual Studio Code is a lightweight editor with strong Git support and useful extensions.

## Windows

- Install via Winget (recommended):

  ```powershell
  winget install --id Microsoft.VisualStudioCode -e
  ```

- Verify:

  ```powershell
  code --version
  ```

- If `code` is not in PATH, launch VS Code, press `Ctrl+Shift+P`, run "Shell Command: Install 'code' command in PATH", then rerun `code --version`.

## macOS

- Install via Homebrew:

  ```bash
  brew install --cask visual-studio-code
  ```

- Verify:

  ```bash
  code --version
  ```

- If `code` is not found, open VS Code, press `Cmd+Shift+P`, and run "Shell Command: Install 'code' command in PATH".

## Linux (Debian/Ubuntu)

- Install Microsoft’s repo and package:

  ```bash
  sudo apt update
  sudo apt install -y wget gpg
  wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
  sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
  sudo apt update
  sudo apt install -y code
  code --version
  ```

- Clean up the downloaded key file if desired:

  ```bash
  rm -f packages.microsoft.gpg
  ```

## Settings Sync with GitHub

I recommend signing into VS Code using your GitHub account as it will allow you to sync your editor settings and profiles so you can have a consistent experience across devices.

## Recommended Settings and Extensions

### Settings

To access your settings, press `Ctrl+Shift+P` on Windows/Linux or `Cmd+Shift+P` on macOS, type "Settings", and select "Preferences: Open Settings (UI)".

- Enable format on save and set a clear font/size:
  - search "format on save"
- Disable folder compaction:
  - search "compact folders"
- Disable file preview:
  - search "workbench editor preview"

### Extensions

VS Code has a huge amount of useful extensions. I've listed some of my recommended extensions below:

- Bash IDE (macOS/Linux, bash language server)
- Powershell (Windows, powershell language server)
- Error Lens (nice error messages)
- markdownlint (formatting for Markdown)
- Prettier - Code formatter (formatter for many filetypes)
- Puppet (for Puppet modules)
- Python (Python support)
- Ruby LSP (Ruby support for working with Puppet)
- YAML (the one from Red Hat, use for YAML support)
