# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`adn-os` is a personal atomic Linux image built with [BlueBuild](https://blue-build.org/) on top of Universal Blue's `ghcr.io/ublue-os/silverblue-main` (Fedora Silverblue, GNOME). It is published to `ghcr.io/adn34/adn-os` and installed on the owner's own machines via `rpm-ostree rebase`. The repo started from the BlueBuild template, so the structure and module names follow BlueBuild conventions.

## Layout

- `recipes/recipe.yml`: the image definition (base image, version tag, ordered module list). Almost all changes go here.
- `files/system/`: copied verbatim into the image root by the `files` module (`etc/`, `usr/`, ...). Currently empty apart from `.gitkeep`.
- `files/scripts/`: shell scripts for the `script` module. Only `example.sh` exists so far.
- `modules/`: custom BlueBuild modules. Currently unused.
- `.github/workflows/build.yml`: CI that builds, signs and publishes the image.
- `cosign.pub`: public key used to verify image signatures. The private key lives in the `SIGNING_SECRET` repo secret. Never commit a private key, and never replace `cosign.pub` without also rotating the secret.

## Current state of the recipe

- Base: `silverblue-main:latest` (follows Universal Blue's Fedora major release; not pinned).
- `dnf` module: installs `micro` and `starship` (`atim/starship` COPR) and removes the Firefox RPM and langpacks.
- `default-flatpaks` module: system Flatpaks from Flathub (Firefox, Loupe, EasyEffects, Refine, Firmware). These are installed on the machine by a service after the rebase and update through normal Flatpak updates; they are not part of the image layers.
- `signing` module: sets up the signature policy needed for `ostree-image-signed`.

## Build and CI

- Builds run weekly (Sundays 06:00 UTC), on every push that touches non-markdown files, on pull requests, and manually through `workflow_dispatch`.
- Scheduled builds are unconditional: they pull the newest base and the newest package versions at that moment. Nothing is pinned unless the recipe pins it.
- Dependabot only bumps GitHub Action versions. It does not track packages.
- Pull request builds are validation builds; only builds from the default branch update `latest`.

## Conventions

- Keep the `# yaml-language-server: $schema=...` header in recipe files so editors validate against the BlueBuild schema.
- Prefer existing BlueBuild modules (`dnf`, `files`, `default-flatpaks`, `systemd`, `script`, ...) over ad-hoc scripts. Use `files/scripts/` only when no module covers the need.
- Check module names and options against the current BlueBuild documentation before adding or changing them. Modules get renamed or replaced between releases, so do not rely on memory.
- Put new config under `files/system/` in the real target path (for example `files/system/etc/...`).
- Update `README.md` when the image contents or build schedule change, so it does not drift from `recipes/recipe.yml` and `build.yml`.

## Working notes

- Explain the mechanism behind a change (why it works, what runs at build time versus on the machine) rather than just the diff.
- Verify suggestions by testing where possible: validate the recipe against the schema, check a package exists in the repos before adding it, and read the CI logs rather than assuming a build passed. Flag anything that could not be tested.
- Correct outdated or context-mismatched information, including anything in this file that no longer matches the repo.
- Ask before pushing to the default branch, changing the workflow, or touching signing files. Small README edits are fine to commit directly.
- Do not add anything private to this repo: it is public, and the published image contains whatever ends up in `files/`.

## References

- BlueBuild docs: https://blue-build.org/
- Module reference (individual pages live under `/reference/modules/<name>/`): https://blue-build.org/reference/module
- Recipe schema (referenced in the `recipe.yml` header): https://schema.blue-build.org/recipe-v1.json
- Upstream base images: https://github.com/ublue-os/main

## Open decisions

- Base image: staying on `silverblue-main` versus moving to a Bazzite-derived base. Revisit before adding hardware- or gaming-specific tweaks.
- Whether to pin `image-version` to a specific Fedora release.
- Whether the `atim/starship` COPR is still needed, or whether `starship` is available in the Fedora repos.
- Where the longer-term customizations should live (services, `files/system` tweaks, extra modules).
