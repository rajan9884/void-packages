# Helium Browser XBPS Package

> NOTE: the per-package `update.sh` was removed. Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging Helium Browser for Void Linux
using the XBPS package manager.

## Files

- `template`: XBPS template file for Helium Browser
- `update`: helper file for `xbps-src update-check` (version detection)
- `update.sh`: script to update the Helium Browser XBPS template
- `files/helium.desktop`: desktop entry for Helium Browser

## Template File

The template file is an XBPS template for Helium Browser.

- Architecture: `x86_64` and `aarch64` (upstream names the arm64 tarball
  `arm64_linux`, mapped from `aarch64*` in the template)
- Build style: precompiled binaries (official upstream tarballs
  `helium-<version>-x86_64_linux.tar.xz` /
  `helium-<version>-arm64_linux.tar.xz`, installed to
  `/opt/helium-browser` with a `/usr/bin/helium-browser` symlink — the same
  pattern as `../zen-browser`, `../zed-editor`, `vivaldi`, `google-chrome`
  and `discord`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

The template file handles the installation of precompiled binaries and
sets up the necessary dependencies.

Current version: `0.18.1.1`.

Notes:

- `CHROME_VERSION_EXTRA` in the bundled `helium-wrapper` is patched to
  `Void Linux` (upstream ships `custom`; Arch/deb/rpm do the same for
  their distro).
- Only `product_logo_256.png` ships upstream, so only the 256x256 hicolor
  icon is installed.
- Upstream also ships `apparmor.cfg` inside `/opt/helium-browser`; it is
  left there (the `.deb` copies it to `/etc/apparmor.d` on install when
  apparmor is present — not done here, same as `vivaldi`/`google-chrome`).

## Installation

To install the Helium Browser package:

Clone the Void Packages repository:

```sh
git clone https://github.com/void-linux/void-packages.git
```

Copy the directory to the `srcpkgs/helium-browser` directory in your Void
Packages repository:

```sh
cp -r helium-browser /path/to/void-packages/srcpkgs/
```

Build the package:

```sh
./xbps-src pkg helium-browser
```

Install the package:

```sh
sudo xbps-install --repository=hostdir/binpkgs helium-browser
```

Run Helium Browser:

```sh
helium-browser
```

Enjoy!

## Update Script

The `update.sh` script automates the process of updating the Helium Browser
XBPS template. It performs the following tasks:

- Fetches the latest release version from the Helium GitHub repository
- Updates the version in the template file (and resets `revision=1`)
- Updates the checksums from the `sha256:` digests published in the
  GitHub release API (no tarball downloads needed)
- Copies the template into your Void Packages checkout
- Builds the updated Helium Browser package
- Installs the updated Helium Browser package

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

To update the Helium Browser package:

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
