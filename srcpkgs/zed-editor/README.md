# Zed XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging Zed for Void Linux using
the XBPS package manager.

## Files

- `template`: XBPS template file for Zed
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the Zed XBPS template
- `files/zed-editor.desktop`: desktop entry for Zed

## Template File

The template file is an XBPS template for Zed.

- Architecture: `x86_64` and `aarch64`
- Build style: precompiled binaries (official upstream tarballs
  `zed-linux-x86_64.tar.gz` / `zed-linux-aarch64.tar.gz`, installed to
  `/opt/zed-editor` with `/usr/bin/zed` and `/usr/bin/zed-editor`
  symlinks — the same pattern as `vivaldi`, `google-chrome` and `discord`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

The template file handles the installation of precompiled binaries and
sets up the necessary dependencies.

Current version: `1.21.0` (upstream tag `v1.21.0`; the template version
drops the leading `v`).

## Installation

To install the Zed package:

Clone the Void Packages repository:

```sh
git clone https://github.com/void-linux/void-packages.git
```

Copy the directory to the `srcpkgs/zed-editor` directory in your Void
Packages repository:

```sh
cp -r zed-editor /path/to/void-packages/srcpkgs/
```

Build the package:

```sh
./xbps-src pkg zed-editor
```

Install the package:

```sh
sudo xbps-install --repository=hostdir/binpkgs zed-editor
```

Run Zed:

```sh
zed
```

Enjoy!

## Update Script

The `update.sh` script automates the process of updating the Zed XBPS
template. It performs the following tasks:

- Fetches the latest release version from the Zed GitHub repository
- Updates the version in the template file (leading `v` stripped,
  `revision=1` reset)
- Updates the checksums from the `sha256:` digests published in the
  GitHub release API (no tarball downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated Zed package
- Installs the updated Zed package

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

To update the Zed package:

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
