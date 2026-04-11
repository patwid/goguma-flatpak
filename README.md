# Goguma Flatpak

Flatpak packaging for [Goguma](https://codeberg.org/emersion/goguma), an IRC client for mobile devices.

Uses [flatpak-flutter](https://github.com/TheAppgineer/flatpak-flutter) to vendor all Dart/pub dependencies for offline, Flathub-compliant builds.

## Prerequisites

- [Docker](https://www.docker.com/) or [Flatpak + flatpak-builder](https://docs.flatpak.org/en/latest/flatpak-builder.html)

## Build Instructions

### 1. Pre-process the manifest

This step clones the goguma source and Flutter SDK, resolves all pub dependencies,
and generates a final manifest with every dependency vendored for offline builds.

```sh
docker run --rm -v "$PWD":/usr/src/flatpak \
  -u $(id -u):$(id -g) \
  theappgineer/flatpak-flutter:0.14.1 flatpak-flutter.yml
```

This produces:

- `fr.emersion.goguma.yml` — the final build manifest
- `generated/modules/flutter-sdk-3.35.7.json` — Flutter SDK module with engine binaries and fonts
- `generated/sources/pubspec.json` — all pub packages as offline source entries

### 2. Build with flatpak-builder

Install the Flatpak runtime and SDK, then build:

```sh
flatpak --user remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak --user install -y flathub org.freedesktop.Platform//24.08 org.freedesktop.Sdk//24.08
flatpak-builder --repo=repo --force-clean --sandbox --user build fr.emersion.goguma.yml
```

The `--sandbox` flag verifies the build works without network access.

<details>
<summary>Using Docker instead of host flatpak-builder</summary>

If you don't have Flatpak set up on the host, you can run the build in a Fedora container:

```sh
docker run --rm --privileged -v "$PWD":/src -w /src fedora:43 bash -c '
  dnf install -y flatpak flatpak-builder &&
  flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo &&
  flatpak --user install -y flathub org.freedesktop.Platform//24.08 org.freedesktop.Sdk//24.08 &&
  flatpak-builder --repo=repo --force-clean --sandbox --user build fr.emersion.goguma.yml
'
```

`--privileged` is required because flatpak-builder's `--sandbox` uses bubblewrap/user namespaces.

</details>

### 3. Run

Run directly from the build directory (no install needed):

```sh
flatpak-builder --run build fr.emersion.goguma.yml goguma
```

Or install and run as a proper Flatpak:

```sh
flatpak --user remote-add --no-gpg-verify --if-not-exists goguma-local repo
flatpak --user install -y goguma-local fr.emersion.goguma
flatpak run fr.emersion.goguma
```

## NixOS

On NixOS, `flatpak-builder` from a devshell is not enough — the Flatpak D-Bus system
helper must be registered. Add the following to your NixOS configuration:

```nix
services.flatpak.enable = true;
xdg.portal.enable = true;
```

Then rebuild (`sudo nixos-rebuild switch`) and the commands in step 2 and 3 will work.

Alternatively, use the Docker approach above which requires no host Flatpak configuration.

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
