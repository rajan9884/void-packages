# Claude Desktop XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging Claude Desktop for Void Linux
using the XBPS package manager. Structure mirrors `../zed-editor`.

## Files

- `template`: XBPS template file for Claude Desktop
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the Claude Desktop XBPS template

No `files/` directory is needed: the upstream `.deb` already ships the
desktop entry (`com.anthropic.Claude.desktop`), icons and copyright file,
which the template installs verbatim.

## Template File

The template file is an XBPS template for Claude Desktop.

- Architecture: `x86_64` and `aarch64` (upstream publishes `amd64`/`arm64`
  `.deb`s; the template maps them to the Void arch names)
- Build style: precompiled binaries (official upstream `.deb`s from
  `https://downloads.claude.ai/claude-desktop/apt/stable`, installed from
  `usr/` verbatim — the same pattern as `google-chrome` and `vivaldi`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

Upstream `.deb` layout (verified against the downloaded package):

- `usr/bin/claude-desktop` → symlink to `../lib/claude-desktop/claude-desktop`
- `usr/lib/claude-desktop/` → Electron app (`chrome-sandbox` ships 4755;
  the template re-applies the suid bit after `vcopy`)
- `usr/share/applications/com.anthropic.Claude.desktop`, icons, docs

Debian maintainer scripts (`postinst` apt-repo registration, AppArmor
profile) are Ubuntu/Debian specific and intentionally not reproduced: on
Void the suid `chrome-sandbox` fallback applies, and updates come from
rebuilding this template, not from an apt repo.

Debian `Depends` → Void mapping used for `depends=` (plus automatic shlib
resolution at build time): `libgtk-3-0`→`gtk+3`, `libnotify4`→`libnotify`,
`libnss3`→`nss`, `libatspi2.0-0`→`at-spi2-core`, `libdrm2`→`libdrm`,
`libgbm1`→`mesa`, `libxcb-dri3-0`→`libxcb`, `libsecret-1-0`→`libsecret`,
`libxtst6`→`libXtst`, `libuuid1`→`libuuid`, plus `alsa-lib` (audio),
`xdg-utils`, `xdg-desktop-portal`, `glib`, `gvfs`, `hicolor-icon-theme`,
`desktop-file-utils`.

Current version: `2.9939.4` (matches the newest `Version:` in both the
`binary-amd64` and `binary-arm64` `Packages` indexes; checksums are the
`SHA256` lines from those indexes).

## Installation

To install the Claude Desktop package:

Clone the Void Packages repository (if you have not already):

```sh
git clone https://github.com/void-linux/void-packages.git ~/.local/share/pkg/void-packages
```

Copy the directory to the `srcpkgs/claude-desktop` directory in your Void
Packages repository:

```sh
cp -r claude-desktop /path/to/void-packages/srcpkgs/
```

Build the package (`restricted=yes` requires explicit opt-in for
proprietary binaries — same as `google-chrome`/`vivaldi`/`discord`):

```sh
XBPS_ALLOW_RESTRICTED=yes ./xbps-src pkg claude-desktop
```

(Permanent alternative, as suggested by the error message itself:
`echo XBPS_ALLOW_RESTRICTED=yes >> etc/conf` from the void-packages root.)

Install the package (password `8858` if sudo prompts):

```sh
sudo xbps-install --repository=hostdir/binpkgs/nonfree claude-desktop
```

Run Claude Desktop:

```sh
claude-desktop
```

Enjoy!

## Optional runtime deps (not hard dependencies)

Anthropic lists these as `Recommends` on Debian; install per need:

- Sign-in persistence: `gnome-keyring` (or `kwallet` on KDE). Without an
  unlocked keyring, sign-in is not saved between launches.
- Cowork tab (VM-backed agentic work): `qemu`, OVMF firmware, `virtiofsd`,
  plus KVM access (`/dev/kvm`) and the `vhost_vsock` kernel module. The
  app reports exactly which piece is missing; restart it after installing.
- Sound: `pulseaudio` (or pipewire, already in most desktops).
- Tray/notification extras: `libappindicator`, `ca-certificates`.

## Update Script

The `update.sh` script automates the process of updating the Claude Desktop
XBPS template. It performs the following tasks:

- Resolves the latest version from the `binary-amd64` APT `Packages` index
  (cross-checked against `binary-arm64`)
- Updates the version in the template file (`revision=1` reset)
- Updates both checksums from the `SHA256` fields in the APT indexes
  (no `.deb` downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated Claude Desktop package
- Installs the updated Claude Desktop package

### Prerequisites

To use the update script, you need:

- xbps-src
- curl
- sed
- awk
- sh
- sudo (only for the install step)

No xtools required (`xgensum`/`xi` are not used).

You must also set the `XBPS_DISTDIR` environment variable to point to
your Void Packages directory.

Example:

```sh
export XBPS_DISTDIR="$HOME/.local/share/pkg/void-packages"
```

Sudo password: the script pipes `$SUDO_PASS` to `sudo -S` for the install
step and defaults it to `8858`. Override without editing the file:

```sh
SUDO_PASS="secret" ./update.sh
```

### Usage

To update the Claude Desktop package:

Ensure you have met all prerequisites, then run the update script:

```sh
./update.sh
```

To bump the template and build without installing:

```sh
./update.sh --skip-install
```

To only bump the template (no build, no install):

```sh
./update.sh --skip-build --skip-install
```

## Contributing

If you want to contribute to this package, please make sure to test your
changes thoroughly before submitting a pull request.

## Issues

If you encounter any issues with the package or the update script,
please open an issue in this repository.
