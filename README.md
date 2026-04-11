# Goguma Flatpak

Flatpak packaging for [Goguma](https://codeberg.org/emersion/goguma), an IRC client for mobile devices.

Uses [flatpak-flutter](https://github.com/TheAppgineer/flatpak-flutter) to vendor all Dart/pub dependencies for offline, Flathub-compliant builds.

## Prerequisites

- [flatpak-builder](https://docs.flatpak.org/en/latest/flatpak-builder.html)
- [Docker](https://www.docker.com/) (recommended) or Python 3.9+ with `packaging`, `pyyaml`, `tomlkit`
- Flathub remote configured:
  ```sh
  flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  ```

## Build Instructions

### 1. Pre-process the manifest

This step clones the goguma source and Flutter SDK, resolves all pub dependencies,
and generates a final manifest with every dependency vendored for offline builds.

Using Docker (recommended):

```sh
docker run --rm -v "$PWD":/usr/src/flatpak \
  -u $(id -u):$(id -g) \
  theappgineer/flatpak-flutter:latest flatpak-flutter.yml
```

Or natively:

```sh
flatpak-flutter.py flatpak-flutter.yml
```

This produces:

- `fr.emersion.goguma.yml` — the final build manifest
- `generated/modules/flutter-sdk-3.29.3.json` — Flutter SDK module with engine binaries and fonts
- `generated/sources/pubspec.json` — all pub packages as offline source entries

### 2. Build with flatpak-builder

```sh
flatpak-builder --repo=repo --force-clean --sandbox --user --install \
  --install-deps-from=flathub build fr.emersion.goguma.yml
```

The `--sandbox` flag verifies the build works without network access.

### 3. Run

```sh
flatpak run fr.emersion.goguma
```

## Patches

- **disable-titlebar.patch** — Removes the GTK header bar and disables window decorations,
  making the app more suitable for mobile form factors (from Alpine aports).

## Files

| File | Purpose |
|------|---------|
| `flatpak-flutter.yml` | Input manifest for flatpak-flutter pre-processing |
| `disable-titlebar.patch` | Disables titlebar for mobile form factor |
| `fr.emersion.goguma.desktop` | Desktop entry |
| `fr.emersion.goguma.metainfo.xml` | AppStream metainfo (required for Flathub) |
