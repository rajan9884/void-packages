# Zen Browser XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging Zen Browser for Void Linux
using the XBPS package manager.

## Files

- `template`: XBPS template file for Zen Browser
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the Zen Browser XBPS template
- `files/zen-browser.desktop`: desktop entry for Zen Browser
- `files/policies.json`: disables the built-in updater (updates are managed
  by `xbps`)

## Template File

The template file is an XBPS template for Zen Browser.

- Architecture: `x86_64` and `aarch64`
- Build style: precompiled binaries (official upstream tarballs
  `zen.linux-x86_64.tar.xz` / `zen.linux-aarch64.tar.xz`, installed to
  `/opt/zen-browser` with a `/usr/bin/zen-browser` symlink — the same
  pattern as `vivaldi`, `google-chrome` and `discord`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

The template file handles the installation of precompiled binaries and
sets up the necessary dependencies.


## Installation

To install the Zen Browser package:

Clone the Void Packages repository:

```sh
git clone https://github.com/void-linux/void-packages.git
```

Copy the directory to the `srcpkgs/zen-browser` directory in your Void
Packages repository:

```sh
cp -r zen-browser /path/to/void-packages/srcpkgs/
```

Build the package:

```sh
./xbps-src pkg zen-browser
```

Install the package:

```sh
sudo xbps-install --repository=hostdir/binpkgs zen-browser
```

Run Zen Browser:

```sh
zen-browser
```

Enjoy!

## Update Script

The `update.sh` script automates the process of updating the Zen Browser
XBPS template. It performs the following tasks:

- Fetches the latest release version from the Zen Browser GitHub repository
- Updates the version in the template file (and resets `revision=1`)
- Updates the checksums from the `sha256:` digests published in the
  GitHub release API (no tarball downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated Zen Browser package
- Installs the updated Zen Browser package

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

To update the Zen Browser package:

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
