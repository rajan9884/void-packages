# WordCraft XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging WordCraft for Void Linux
using the XBPS package manager. Structure mirrors `../claude-desktop`
(which itself mirrors `../zed-editor`).

## Files

- `template`: XBPS template file for WordCraft
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the WordCraft XBPS template

No `files/` directory is needed: the upstream `.deb` already ships the
desktop entry (`ai.storyteller.wordcraft.desktop`), icons, MIME info,
metainfo and license files, which the template installs verbatim.

## Template File

The template file is an XBPS template for WordCraft.

- Architecture: `x86_64` and `aarch64` (upstream publishes
  `linux-x86_64`/`linux-aarch64` `.deb`s on GitHub releases; the names
  already match the Void arch names, so no mapping is needed)
- Build style: precompiled binaries (official upstream `.deb`s from
  `https://github.com/storytold/wordcraft/releases`, installed from
  `usr/` verbatim — the same pattern as `google-chrome` and `vivaldi`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

Upstream `.deb` layout (verified against the downloaded package):

- `usr/bin/wordcraft`, `usr/bin/wordcraft-cli` (statically linked Rust
  binaries; only `libgcc`/`libc` are `NEEDED`, the rest is dlopen'd)
- `usr/share/applications/ai.storyteller.wordcraft.desktop`, icons,
  MIME package, metainfo, docs (`LICENSE-MIT`, `LICENSE-APACHE`)

Debian `Depends` → Void mapping used for `depends=` (plus automatic shlib
resolution at build time): `libxkbcommon0`→`libxkbcommon`,
`libvulkan1`→`vulkan-loader` (GPU-accelerated rendering; dlopen'd, so not
auto-detected), plus `hicolor-icon-theme` and `desktop-file-utils` for the
shipped desktop entry/icons (same call as the AUR `wordcraft-bin`
package, which depends on `libxkbcommon`, `vulkan-icd-loader`,
`hicolor-icon-theme`, `gcc-libs`, `glibc`).

Current version: `0.6.1` (upstream tag `v0.6.1`; the template version
drops the leading `v`. Checksums are the `sha256:` digests from the
GitHub release API, matching `SHA256SUMS.txt`).

## Installation

To install the WordCraft package:

Clone the Void Packages repository (if you have not already):

```sh
git clone https://github.com/void-linux/void-packages.git ~/.local/share/pkg/void-packages
```

Copy the directory to the `srcpkgs/wordcraft` directory in your Void
Packages repository:

```sh
cp -r wordcraft /path/to/void-packages/srcpkgs/
```

Build the package:

```sh
./xbps-src pkg wordcraft
```

Install the package:

```sh
sudo xbps-install --repository=hostdir/binpkgs wordcraft
```

Run WordCraft:

```sh
wordcraft
```

Enjoy!

## Update Script

The `update.sh` script automates the process of updating the WordCraft
XBPS template. It performs the following tasks:

- Resolves the latest version from the GitHub `releases/latest` redirect
  (leading `v` stripped)
- Updates the version in the template file (`revision=1` reset)
- Updates both checksums from the `sha256:` digests published in the
  GitHub release API (no `.deb` downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated WordCraft package
- Installs the updated WordCraft package

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

### Usage

To update the WordCraft package:

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
