# Publishing on Flathub

Guide for submitting and maintaining `fr.emersion.goguma` on [Flathub](https://flathub.org/).

## Prerequisites

Before submitting, verify the build and metadata are correct.

### Validate AppStream metadata

```sh
appstreamcli validate fr.emersion.goguma.metainfo.xml
```

The metainfo file must include at minimum:

- App ID, name, summary, and description
- License (`GPL-3.0-or-later`) and metadata license (`CC0-1.0`)
- Developer information
- At least one `<release>` entry
- Content rating
- Homepage and bug tracker URLs

### Validate the desktop file

```sh
desktop-file-validate fr.emersion.goguma.desktop
```

### Verify the offline build

The build must succeed without network access. Run with `--sandbox`:

```sh
flatpak-builder --repo=repo --force-clean --sandbox --user build fr.emersion.goguma.yml
```

## Submission

1. **Fork** [flathub/flathub](https://github.com/flathub/flathub) on GitHub.

2. **Create a new branch** named `fr.emersion.goguma`.

3. **Add the following files** to the branch root:
   - `fr.emersion.goguma.yml` — the final manifest
   - `fr.emersion.goguma.desktop` — desktop entry
   - `fr.emersion.goguma.metainfo.xml` — AppStream metadata
   - `disable-titlebar.patch` — patch file
   - `generated/modules/flutter-sdk-3.29.3.json` — Flutter SDK module
   - `generated/sources/pubspec.json` — vendored pub dependencies

4. **Open a pull request** against `flathub/flathub` with a brief description of the app.

5. **Wait for review.** A Flathub maintainer will check:
   - `finish-args` permissions are minimal and justified
   - AppStream metadata is complete (description, screenshots, releases)
   - All sources are vendored (no network access during build)
   - Desktop file and icons are properly installed
   - The app follows [Flathub quality guidelines](https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/)

## What to Expect During Review

Common feedback areas:

- **Permissions:** Each `finish-args` entry may be questioned. Current permissions
  (`--share=network`, `--share=ipc`, `--socket=wayland`, `--socket=fallback-x11`,
  `--device=dri`, `--talk-name=org.freedesktop.Notifications`) are reasonable for
  a networked GUI app with notification support.

- **Screenshots:** Flathub requires at least one screenshot in the metainfo file.
  Images should be hosted at a stable URL (e.g., in the upstream repo).

- **Icon size:** Flathub recommends a 128x128 or larger app icon. The current manifest
  installs up to 192x192, which should be sufficient.

## After Approval

Once the PR is merged:

- Flathub creates a dedicated repo at `https://github.com/flathub/fr.emersion.goguma`.
- The app is built by Flathub's infrastructure and published automatically.
- Future updates are submitted as PRs to that repo (or pushed directly if you have maintainer access).

### Updating the App

To publish a new version:

1. Re-run the flatpak-flutter pre-processor to update the generated sources:
   ```sh
   docker run --rm -v "$PWD":/usr/src/flatpak \
     -u $(id -u):$(id -g) \
     theappgineer/flatpak-flutter:latest flatpak-flutter.yml
   ```

2. Update the `tag` and `commit` in `flatpak-flutter.yml` to the new release.

3. Add a new `<release>` entry to `fr.emersion.goguma.metainfo.xml`.

4. Commit and push to the `flathub/fr.emersion.goguma` repo (or open a PR).

## References

- [Flathub Submission Guide](https://docs.flathub.org/docs/for-app-authors/submission)
- [Flathub Metainfo Guidelines](https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/)
- [AppStream Documentation](https://www.freedesktop.org/software/appstream/docs/)
- [Flatpak Manifest Reference](https://docs.flatpak.org/en/latest/manifests.html)
