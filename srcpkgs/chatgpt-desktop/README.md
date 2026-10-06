# ChatGPT Desktop XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging ChatGPT Desktop for Void Linux
using the XBPS package manager. Structure mirrors `../claude-desktop`
(which itself mirrors `../zed-editor`).

## Files

- `template`: XBPS template file for ChatGPT Desktop
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the ChatGPT Desktop XBPS template

No `files/` directory is needed: the upstream `.deb` already ships the
desktop entry (`chatgpt.desktop`), icons and copyright file, which the
template installs verbatim.

## Template File

The template file is an XBPS template for ChatGPT Desktop.

- Architecture: `x86_64` and `aarch64` (upstream publishes `amd64`/`arm64`
  `.deb`s; the template maps them to the Void arch names)
- Build style: precompiled binaries (official upstream `.deb`s from
  `https://persistent.oaistatic.com/codex-app-prod/linux/deb`, installed from
  `usr/` verbatim — the same pattern as `google-chrome` and `vivaldi`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

Upstream `.deb` layout (verified against the downloaded package):

- `usr/bin/chatgpt` → symlink to `../lib/chatgpt/codex-launcher`
  (a shell wrapper that `exec`s `ChatGPT`, so CLI flags pass through)
- `usr/lib/chatgpt/` → Electron app (no `chrome-sandbox` binary is shipped;
  sandboxing relies on unprivileged user namespaces, same as on Ubuntu)
- `usr/share/applications/chatgpt.desktop`, icons, docs

Debian maintainer scripts (`postinst` apt-repo registration + keyring,
AppArmor profile under `etc/`) are Ubuntu/Debian specific and intentionally
not reproduced: `post_extract` drops `etc/`, and updates come from
rebuilding this template, not from an apt repo (same call as
`claude-desktop`).

Debian `Depends` → Void mapping used for `depends=` (plus automatic shlib
resolution at build time): `libgtk-3-0`→`gtk+3`, `libnotify4`→`libnotify`,
`libnss3`→`nss`, `libatspi2.0-0`→`at-spi2-core`, `libdrm2`→`libdrm`,
`libgbm1`→`mesa`, `libxcb-dri3-0`→`libxcb`, `libsecret-1-0`→`libsecret`,
`libxtst6`→`libXtst`, `libuuid1`→`libuuid`, `libcups2`→`cups`,
`libxkbcommon0`→`libxkbcommon`, plus `alsa-lib` (audio), `xdg-utils`,
`xdg-desktop-portal`, `glib`, `gvfs`, `hicolor-icon-theme`,
`desktop-file-utils`.

Current version: `26.930.21537` (matches the newest `Version:` in both the
`binary-amd64` and `binary-arm64` `Packages` indexes; checksums are the
`SHA256` lines from those indexes).

## Installation

To install the ChatGPT Desktop package:

Clone the Void Packages repository (if you have not already):

```sh
git clone https://github.com/void-linux/void-packages.git ~/.local/share/pkg/void-packages
```

Copy the directory to the `srcpkgs/chatgpt-desktop` directory in your Void
Packages repository:

```sh
cp -r chatgpt-desktop /path/to/void-packages/srcpkgs/
```

Build the package (`restricted=yes` requires explicit opt-in for
proprietary binaries — same as `google-chrome`/`vivaldi`/`discord`):

```sh
XBPS_ALLOW_RESTRICTED=yes ./xbps-src pkg chatgpt-desktop
```

(Permanent alternative, as suggested by the error message itself:
`echo XBPS_ALLOW_RESTRICTED=yes >> etc/conf` from the void-packages root.)

Install the package (password `8858` if sudo prompts):

```sh
sudo xbps-install --repository=hostdir/binpkgs/nonfree chatgpt-desktop
```

Run ChatGPT Desktop:

```sh
chatgpt
```

Enjoy!

## Sign-in persistence (sway / unrecognized desktops)

Same Electron/keyring quirk as `claude-desktop`: on desktops Chromium does
not recognize (e.g. `XDG_CURRENT_DESKTOP=sway:wlroots:swayfx`), it silently
falls back to plaintext storage even with an unlocked `gnome-keyring`, and
the app warns that sign-in won't be saved. Force the libsecret store with a
user-local launcher override (survives package updates):

```sh
mkdir -p ~/.local/share/applications
cp /usr/share/applications/chatgpt.desktop ~/.local/share/applications/
sed -i 's|^Exec=chatgpt |Exec=chatgpt --password-store=gnome-libsecret |' \
  ~/.local/share/applications/chatgpt.desktop
update-desktop-database ~/.local/share/applications/
```

Then sign in once more — the session persists across restarts after that.

## Optional runtime deps (not hard dependencies)

OpenAI lists these as `Recommends` on Debian; install per need:

- Sign-in persistence: `gnome-keyring` (or `kwallet` on KDE). Without an
  unlocked keyring, sign-in is not saved between launches (see above).
- Sound: `pulseaudio` (or pipewire, already in most desktops).
- `git` (used by Codex features).

## Update Script

The `update.sh` script automates the process of updating the ChatGPT
Desktop XBPS template. It performs the following tasks:

- Resolves the latest version from the `binary-amd64` APT `Packages` index
  (cross-checked against `binary-arm64`)
- Updates the version in the template file (`revision=1` reset)
- Updates both checksums from the `SHA256` fields in the APT indexes
  (no `.deb` downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated ChatGPT Desktop package
- Installs the updated ChatGPT Desktop package

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

To update the ChatGPT Desktop package:

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
