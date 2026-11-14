# WaveR pacman repository

Self-hosted [pacman](https://wiki.archlinux.org/title/Pacman) repository for
[WaveR](https://github.com/FRANORDE/WaveR), a simple, cozy local music player.

Packages are built automatically from the WaveR releases (see
[`.github/workflows/pacman-repo.yml`](.github/workflows/pacman-repo.yml)) and
published here on the [`gh-pages`](../../tree/gh-pages) branch. New WaveR
releases are picked up within 6 hours, with no manual steps. The repository
always contains the latest release only; older versions remain available as
[WaveR release assets](https://github.com/FRANORDE/WaveR/releases).

## Install

Add the repository to `/etc/pacman.conf`:

```ini
[wave-r]
Server = https://franorde.github.io/pacman-repo/x86_64
SigLevel = Optional TrustAll
```

Then install the package:

```bash
sudo pacman -Syu wave-r
```

Runtime dependencies (`alsa-lib`, `gtk3`) are installed automatically, and new
releases are picked up by a regular `pacman -Syu`.

## Uninstall

```bash
sudo pacman -R wave-r
```

## Why `SigLevel = Optional TrustAll`?

Packages in this repository are currently not GPG-signed. The line above tells
pacman to accept unsigned databases and packages, but **only for this
repository** — your system-wide signature policy stays untouched. If you
prefer strictness, remove the line: pacman will then simply refuse the
repository.

## How this repository works

- The [`pacman-repo`](.github/workflows/pacman-repo.yml) workflow runs in an
  `archlinux:base-devel` container.
- It downloads the `WaveR_<version>_x86_64.tar.gz` payload from the
  [WaveR releases](https://github.com/FRANORDE/WaveR/releases), wraps it into
  a proper Arch package (`wave-r`) with `makepkg`, indexes it with `repo-add`
  and pushes the result to `gh-pages`.
- It is triggered **automatically every 6 hours**: a cheap check job compares
  the latest *published* WaveR release with the `latest` marker on `gh-pages`
  and publishes only when they differ (draft releases are ignored, and a
  manual dispatch with an explicit version always publishes):

  ```bash
  gh workflow run pacman-repo.yml            # no input = check the latest release
  gh workflow run pacman-repo.yml -f version=2.0.0   # force a rebuild
  ```

  The WaveR release must be **published** first (draft assets are not
  downloadable without authentication).
