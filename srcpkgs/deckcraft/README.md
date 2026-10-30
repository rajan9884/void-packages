# DeckCraft XBPS Package

> NOTE: Use `vp-sync` at the void-packages root (branch `custom`) to check/bump/build/install all custom templates in one command (e.g. `vp-sync`, `vp-sync --check-only`).

This repository contains files for packaging DeckCraft for Void Linux
using the XBPS package manager. Structure mirrors `../wordcraft` and `../claude-desktop`.

## Files

- `template`: XBPS template file for DeckCraft
- `update`: helper file for `xbps-src update-check` (version detection)

No `files/` directory is needed: the upstream `.deb` already ships the
desktop entry (`ai.storyteller.deckcraft.desktop`), icons, MIME info,
metainfo and license files, which the template installs verbatim.

## Template File

The template file is an XBPS template for DeckCraft.

- Architecture: `x86_64` and `aarch64` (upstream publishes
  `linux-x86_64`/`linux-aarch64` `.deb`s on GitHub releases; the names
  already match the Void arch names, so no mapping is needed)
- Build style: precompiled binaries (official upstream `.deb`s from
  `https://github.com/storytold/deckcraft/releases`, installed from
  `usr/` verbatim — the same pattern as `google-chrome` and `vivaldi`)
- Maintainer: Rajan Jaiswal <rajan.dev.jaiswal@gmail.com>

Upstream `.deb` layout (verified against the downloaded package):

- `usr/bin/deckcraft`, `usr/bin/deckcraft-cli`
- `usr/share/applications/ai.storyteller.deckcraft.desktop`, icons,
  MIME package, metainfo, docs (`LICENSE-MIT`, `LICENSE-APACHE`)

Debian `Depends` → Void mapping used for `depends=` (plus automatic shlib
resolution at build time): `libxkbcommon0`→`libxkbcommon`,
`libvulkan1`→`vulkan-loader` (GPU-accelerated rendering; dlopen'd, so not
auto-detected), `libasound2`→`alsa-lib`, plus `hicolor-icon-theme` and
`desktop-file-utils` for the shipped desktop entry/icons.

Current version: `0.12.0` (upstream tag `v0.12.0`; the template version
drops the leading `v`. Checksums are the `sha256:` digests from the
GitHub release API, matching `SHA256SUMS.txt`).

## Installation

To build and install the package using `vp-sync`:

```sh
./vp-sync deckcraft
```

Or manually via `xbps-src`:

```sh
./xbps-src pkg deckcraft
sudo xbps-install --repository=hostdir/binpkgs deckcraft
```

Run DeckCraft:

```sh
deckcraft
```
