# sandbutter

**sandbutter** is a lightweight, zero-cost Copy-on-Write (CoW) sandbox manager for Linux. It creates instant clones of your running host operating system using **Btrfs snapshots** and drops you into isolated, fully functional container environments using **`systemd-nspawn`**.

Because sandboxes leverage Btrfs subvolume snapshots, creating a sandbox takes milliseconds and consumes **0 additional disk bytes** upfront. Changes inside the sandbox are isolated and copy-on-write, protecting your host system while retaining full parity with your installed software, dotfiles, user configuration, and system libraries.

---

## Features

- ⚡ **Instant Zero-Cost Clones**: Create complete host OS sandboxes in milliseconds via Btrfs CoW snapshots.
- 🐧 **Full Host Parity**: Mirrors your running environment, packages, dotfiles, and users inside the sandbox.
- 🐚 **Custom Shell Support**: Launch directly into your preferred shell (`bash`, `zsh`, `fish`, `tmux`, etc.) or custom commands.
- 💨 **Ephemeral Sessions**: Spin up temporary disposable containers where all changes are discarded on exit (`sandbutter ephemeral`).
- 🔄 **Reflink Data Exchange**: Zero-copy data sharing between sandbox and host via `pull` and `push` with Btrfs extent reflinking.
- 🔍 **In-Sandbox Diffing**: Compare files or directories between the host and sandbox with `sandbutter diff`.
- 🧩 **Multi-Btrfs Layout Adaptation**:
  - Accommodates single subvolumes, standard `@` / `@home` / `@root` split layouts (Debian, Ubuntu, Fedora, Arch), and complex multi-subvolume layouts (openSUSE `/var`, `/opt`, `/usr/local`).
  - Safely excludes ephemeral and virtualization paths (`/.snapshots`, `/tmp`, `/var/log`, container pools).
  - Gracefully handles separate `/home` filesystems with automatic user environment provisioning.
- 🩺 **Built-in Diagnostics**: Run `sandbutter check` to verify kernel, packages, filesystem UUIDs, and subvolume compatibility.
- 🛡️ **Strict Safety Guardrails**: Path sanitization and containment checks prevent accidental host data loss or traversal.

---

## Requirements

sandbutter requires Linux with a **Btrfs root filesystem** and the following packages:

| Package | Purpose | Debian / Ubuntu | Fedora / RHEL | Arch Linux | openSUSE |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **btrfs-progs** | Subvolume management & snapshots | `btrfs-progs` | `btrfs-progs` | `btrfs-progs` | `btrfs-progs` |
| **systemd-container** | Container execution (`systemd-nspawn`) | `systemd-container` | `systemd-container` | `systemd` | `systemd-container` |
| **util-linux** | Mount and layout inspection (`findmnt`) | `util-linux` | `util-linux` | `util-linux` | `util-linux` |
| **coreutils** | File operations with reflink support | `coreutils` | `coreutils` | `coreutils` | `coreutils` |

### Package Installation

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install -y btrfs-progs systemd-container

# Fedora / RHEL
sudo dnf install -y btrfs-progs systemd-container

# Arch Linux
sudo pacman -S --needed btrfs-progs systemd

# openSUSE
sudo zypper install -y btrfs-progs systemd-container
```

---

## Installation

Clone the repository and make the script executable:

```bash
git clone <repo-url> sandbutter
cd sandbutter
chmod +x sandbutter

# Optional: Symlink or install to your PATH
sudo install -m 755 sandbutter /usr/local/bin/sandbutter
```

---

## Quick Start

### 1. Check System Compatibility

Verify that your system meets all requirements:

```bash
sandbutter check
```

### 2. Create a Sandbox

Create a new sandbox named `dev-test`:

```bash
sandbutter create dev-test
```

### 3. Enter the Sandbox

Drop into an interactive shell as your regular user:

```bash
# Default user login shell
sandbutter enter dev-test

# Specify a custom shell (positional or flag)
sandbutter enter dev-test zsh
sandbutter enter dev-test -s /bin/bash
```

Or drop into an interactive root shell:

```bash
sandbutter root dev-test zsh
```

### 4. Run an Ephemeral (Disposable) Session

Run a session where any modified packages or files are discarded on exit:

```bash
sandbutter ephemeral dev-test
```

### 5. Transfer Data via Zero-Cost Reflink

```bash
# Reflink copy a built artifact from sandbox to host (0 disk bytes written)
sandbutter pull dev-test /home/user/project/dist/app ./dist/app

# Reflink copy a host file into the sandbox
sandbutter push dev-test ./config.json /home/user/project/config.json

# Diff changes between host and sandbox
sandbutter diff dev-test /etc/hosts
```

### 6. Inspect and Delete

```bash
# List all sandboxes
sandbutter list

# View subvolume details and nested child subvolumes
sandbutter status dev-test

# Recursively delete the sandbox and reclaim extents
sandbutter delete dev-test
```

---

## Command Reference

```text
Usage: sandbutter <command> [arguments]

Lifecycle Commands:
  create <name>            Create a new CoW host snapshot sandbox
  enter <name> [shell]     Drop into interactive shell as your regular user
  root <name> [shell]      Drop into interactive shell as root
  ephemeral <name> [shell] Run temporary session (changes discarded on exit)
  list                     List all existing sandboxes
  status <name>            Inspect subvolume details and extent state
  delete <name>            Delete sandbox subvolumes and reclaim dirty extents

Reflink / Data Commands:
  pull <name> <path> [dst] Reflink-copy file/dir from sandbox to host (zero bytes written)
  push <name> <path> [dst] Reflink-copy file/dir from host into sandbox
  diff <name> <path> [dst] Diff file/dir between host and sandbox

Diagnostic Commands:
  check                    Verify system packages and Btrfs filesystem support
```

---

## Configuration

- **`SANDBUTTER_MACHINES_DIR`**: Path where sandboxes are stored (default: `/var/lib/machines`). Must reside on the same Btrfs filesystem as `/` to allow instant CoW snapshots.
- **`TARGET_USER`**: Target non-root user when using `enter` or `ephemeral` (default: `$SUDO_USER` or `$USER`).

---

## License

This project is licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
