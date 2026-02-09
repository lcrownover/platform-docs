# Command-Line Editors

When you're working on a remote server over SSH, you need a terminal-based text editor. Two common choices are **nano** (simple and beginner-friendly) and **vim** (powerful but has a learning curve).

Pick whichever you're comfortable with. If you've never used either, start with nano.

## Nano

Nano is the easiest editor to pick up. It works like a basic text editor: you type, you move around with arrow keys, and the available commands are listed at the bottom of the screen.

### Open a File

```bash
nano /path/to/file.conf
```

### Save and Exit

1. Press `Ctrl+O` to **write out** (save) the file
2. Press `Enter` to confirm the filename
3. Press `Ctrl+X` to **exit**

If you want to exit without saving, press `Ctrl+X`, then `N` when it asks whether to save.

!!! tip
    The `^` symbol in nano's bottom bar means `Ctrl`. So `^O` means `Ctrl+O`.

### Installation

Nano is pre-installed on most Linux distributions.

## Vim

Vim is a modal editor, meaning it has different modes for inserting text and running commands. If you already know modal editing, vim is extremely efficient. If you don't, nano is a better starting point for lab work.

### Open a File

```bash
vim /path/to/file.conf
```

### Survival Basics

| Action | Keys |
|--------|------|
| Enter insert mode (start typing) | `i` |
| Exit insert mode | `Esc` |
| Save | `:w` then `Enter` |
| Quit | `:q` then `Enter` |
| Save and quit | `:wq` then `Enter` |
| Quit without saving | `:q!` then `Enter` |

### Installation

Vim is pre-installed on most Linux distributions. If only the minimal `vi` is available:

```bash
# Ubuntu/Debian
sudo apt install -y vim

# RHEL/Rocky
sudo dnf install -y vim-enhanced
```
