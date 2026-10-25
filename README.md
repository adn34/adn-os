# adn-os &nbsp; [![bluebuild build badge](https://github.com/adn34/adn-os/actions/workflows/build.yml/badge.svg)](https://github.com/adn34/adn-os/actions/workflows/build.yml)

My personal atomic Linux image, built with [BlueBuild](https://blue-build.org/) on top of [Universal Blue](https://universal-blue.org/)'s `silverblue-main` (Fedora Silverblue with GNOME). It is published to `ghcr.io/adn34/adn-os`.

This is a personal project, built for my own machines. It is public so the recipe is easy to read and reuse, but there are no stability or support guarantees.

## What's in it

The image is intentionally close to stock for now. Changes on top of the base:

- **Packages (RPM):** `micro` and `starship` are added; `starship` comes from the `atim/starship` COPR.
- **Removed:** the Firefox RPM and its langpacks, replaced by the Flatpak.
- **Default Flatpaks (system scope, from Flathub):** Firefox, Loupe, EasyEffects, Refine, Firmware. These are installed on the machine by a service after the rebase, not baked into the image, and they update through the normal Flatpak mechanism.
- **System files:** anything under `files/system/` is copied into the image root (currently empty).
- **Signing:** images are signed with cosign (see [Verification](#verification)).

The full definition is in [`recipes/recipe.yml`](recipes/recipe.yml).

## Builds

The GitHub Actions workflow in [`.github/workflows/build.yml`](.github/workflows/build.yml) builds and publishes the image:

- **Weekly**, on Sundays at 06:00 UTC, after the Universal Blue base images have been rebuilt. Each scheduled build starts from the newest base and installs the newest package versions available at that moment; nothing is pinned.
- **On every push** that touches more than documentation.
- **On pull requests** (built but not published as `latest`).
- **Manually**, using the *Run workflow* button, for an update between scheduled builds.

`image-version` in the recipe is `latest`, so the image follows Universal Blue when it moves to a new Fedora major release. Pin it to a specific version in `recipes/recipe.yml` to stay on a given release.

## Installation

> [!WARNING]
> [This is an experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable), use at your own discretion.

To rebase an existing atomic Fedora installation to the latest build:

1. First rebase to the unsigned image, to get the proper signing keys and policies installed:
   ```
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/adn34/adn-os:latest
   ```
2. Reboot to complete the rebase:
   ```
   systemctl reboot
   ```
3. Then rebase to the signed image:
   ```
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/adn34/adn-os:latest
   ```
4. Reboot again to complete the installation:
   ```
   systemctl reboot
   ```

The `latest` tag always points to the most recent build. To go back to the previous deployment, run `rpm-ostree rollback` and reboot.

## ISO

If you build on Fedora Atomic, you can generate an offline ISO by following the [BlueBuild instructions](https://blue-build.org/how-to/generate-iso/). ISOs are too large to host on GitHub for free, so none are published here.

## Verification

Images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). To verify the signature, download `cosign.pub` from this repo and run:

```bash
cosign verify --key cosign.pub ghcr.io/adn34/adn-os
```
