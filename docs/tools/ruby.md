# Ruby

Ruby is required for several Puppet ecosystem tools including [puppet-lint](puppet-lint.md). You don't need to write Ruby code, but you need a working Ruby installation where `gem install` works and installed gems are in your `PATH`.

## Installation

### macOS

**Homebrew** (recommended)

macOS ships with a system Ruby, but it's outdated and requires `sudo` to install gems. Use Homebrew instead:

```bash
brew install ruby
```

Open a new terminal window, and run

```bash
which ruby
```

If the displayed path does not include `homebrew`, or you receive any errors, you'll need to add the Homebrew Ruby and its gem binaries to your `PATH`. 
Add these lines to your shell config (`~/.zshrc` for zsh, `~/.bashrc` for bash):

```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
export PATH="$(gem environment gemdir)/bin:$PATH"
```

Reload your shell:

```bash
source ~/.zshrc   # or source ~/.bashrc
```

!!! warning
    If you skip the `PATH` step, your shell will use the system Ruby and `gem install` will either fail or install gems somewhere you can't run them.

### Windows

Download and run the installer from [rubyinstaller.org](https://rubyinstaller.org/downloads/). Choose the version marked **(x64) WITH DEVKIT** -- the devkit is needed to install gems that compile native extensions.

During installation:

- You may be prompted with a Windows Security warning, click "More info", then the "Run anyway" button
- For the install mode, click "Install for me only (recommended)"
- Leave the default install path (`C:\Ruby34-x64` or similar)
- Ensure **"Add Ruby executables to your PATH"** and **"Associate .rb and .rbw files with this installation"** are checked
- On the final screen, leave **"Run 'ridk install'"** checked and press Finish
- In the MSYS2 installer that opens, press Enter to run the default option. Once that's finished, press Enter again to exit that installer

Open a new Command Prompt or PowerShell window and verify:

```
ruby -v
gem -v
```

RubyInstaller adds both Ruby and gem binaries to your `PATH` automatically. Gems installed with `gem install` should be runnable immediately in any new terminal window.

!!! tip
    If you're also using [WSL](wsl.md), you'll need to install Ruby separately inside WSL -- the Windows installation won't be available there. Follow the Linux instructions below for WSL.

### Linux (Ubuntu) / WSL

```bash
sudo apt update && sudo apt install -y ruby ruby-dev
```

Ubuntu's packaged Ruby installs gems to a user directory by default, but you need to make sure that directory is in your `PATH`. 

Add this to your `~/.bashrc`, replacing `4.x.x` with the proper version that you installed (discoverable with `ruby -v`):

```bash
export PATH="$HOME/.local/share/gem/ruby/4.x.x/bin:$PATH"
```

Reload your shell:

```bash
source ~/.bashrc
```

## Verify Installation

```bash
ruby -v
gem -v
```

You should see version numbers for both.

## Verify Gems Are in PATH

Install a gem and confirm it's runnable:

```bash
gem install puppet-lint
puppet-lint --version
```

If `puppet-lint --version` works, your `PATH` is set up correctly. If you get "command not found," revisit the `PATH` steps above.

## Troubleshooting

### "You don't have write permissions" on gem install

You're using the system Ruby. On macOS, install Ruby via Homebrew and update your `PATH`. On Linux, gems should install to your home directory by default; if they don't, add `--user-install` to your gem commands:

```bash
gem install --user-install puppet-lint
```

### Gem installs but "command not found" when running it

The gem's `bin` directory isn't in your `PATH`. Find where gems are installed:

```bash
gem environment gemdir
```

Add the `bin` subdirectory of that path to your `PATH` in your shell config.
