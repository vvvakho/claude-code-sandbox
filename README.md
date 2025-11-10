# Claude Code Sandbox

A macOS sandbox configuration for Claude Code that restricts filesystem READ access for enhanced security.

Just about all sandbox-exec attempts for `claude` you'll find on Github allow full filesystem read access and full network access – all they protect against is file overwrites. But that's not very secure - prompt injection could leak data from all over your filesystem if `claude` can read it.

## Overview

This project provides a macOS `sandbox-exec` profile that limits `claude`'s access to your filesystem. It prevents Claude Code from reading your home directory (except for the current working directory) and restricts writes to only the target directory and temporary locations.

## Features

- **Restricted Read Access**: Blocks reading from file system except for:
  - Current working directory (`TARGET_DIR`)
  - Git configuration files (`.gitconfig`, `.config/git`)
  - System directories (/usr, /bin, /opt, /var, /private/var, /nix)
  - It allows _listing_ directories leading up to `TARGET_DIR`, because otherwise claude will glitch.
    However, even though files in `~` can be listed with `ls` by claude, they (and their metadata) cannot be read

- **Restricted Write Access**: Only allows writing to:
  - Current working directory (`TARGET_DIR`)
  - Temporary directories (`/tmp`, `/var/folders`)
  - Cache directory (`~/.cache`)
  - Claude configuration (`~/.claude`)

- **Network Access**: Full network access enabled (required for Claude API)
- **Keychain Access**: Allows reading from macOS Keychain for API key storage

## Installation

Run the installation script:

```bash
./install
```

This will:
1. Copy the sandbox profile to `~/.config/claude-sandbox/noread.sb`
2. Install the `claude-sandbox` wrapper script to `~/.local/bin/claude-sandbox`
3. Make the wrapper executable

Ensure `~/.local/bin` is in your PATH.

## Usage

Instead of running `claude` directly, use the `claude-sandbox` wrapper:

```bash
# run bash to browse the sandbox-accessible filesystem
~/.local/bin/claude-sandbox bash

# run claude
~/.local/bin/claude-sandbox claude
```

## How It Works

The `claude-sandbox` wrapper uses macOS's `sandbox-exec` command to apply a security profile defined in `noread.sb`. This sandbox profile is based on the [Para sandboxing profile](https://github.com/2mawi2/para/blob/218259b6e260be43334f308a74108f31920f7ca4/src/core/sandbox/profiles/standard.sb) and [anthropic's sandbox-runtime's dynamic profile](https://github.com/anthropic-experimental/sandbox-runtime/blob/1bafa66a2c3ebc52569fc0c1a868e85e778f66a0/src/sandbox/macos-sandbox-utils.ts#L200), with additional configuration for Claude Code compatibility.
